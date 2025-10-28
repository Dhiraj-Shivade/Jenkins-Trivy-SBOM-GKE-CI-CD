pipeline {
  agent {
    node {
      label 'slave'
      customWorkspace '/home/dhiraj_shivade1/python-app'
    }
  }

  environment {
    PROJECT_ID = "clear-booking-470907-q6"
    REGION = "asia-south1"
    REPO_NAME = "python-app-repo"
    IMAGE = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/python-app"
    CLUSTER = "cluster-ci"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Auth to GCP') {
      steps {
        withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'SA_KEY_FILE')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$SA_KEY_FILE
            gcloud config set project ${PROJECT_ID}
            gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
          '''
        }
      }
    }

    stage('Verify Docker') {
      steps {
        sh '''
          command -v docker && docker --version
          docker ps || true
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          docker build -t ${IMAGE}:$BUILD_NUMBER .
          docker tag ${IMAGE}:$BUILD_NUMBER ${IMAGE}:latest
        '''
      }
    }

    stage('Trivy SBOM Scan') {
      steps {
        sh '''
          # Scan + generate vulnerability report table
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy:latest image \
            --scanners vuln \
            --format table \
            ${IMAGE}:$BUILD_NUMBER | tee trivy-report.txt

          # Generate CycloneDX SBOM
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v $WORKSPACE:/result \
            aquasec/trivy:latest image \
            --format cyclonedx \
            --scanners vuln \
            --output /result/sbom.cyclonedx.json \
            ${IMAGE}:$BUILD_NUMBER
        '''
      }
      post {
        always {
          archiveArtifacts artifacts: 'sbom.cyclonedx.json, trivy-report.txt', fingerprint: true
        }
      }
    }

    stage('Push to Artifact Registry') {
      steps {
        sh '''
          docker push ${IMAGE}:$BUILD_NUMBER
          docker push ${IMAGE}:latest
        '''
      }
    }

    stage('Deploy to GKE') {
      steps {
        sh '''
          gcloud container clusters get-credentials ${CLUSTER} --region ${REGION} --project ${PROJECT_ID}
          
          kubectl apply -f k8s/service-clusterip.yaml

          kubectl set image deployment/python-app-deploy \
            python-app-container=${IMAGE}:$BUILD_NUMBER --record

          kubectl rollout status deployment/python-app-deploy --timeout=180s
        '''
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed! Check logs & downloaded Trivy SBOM artifacts."
    }
    success {
      echo "✅ Deployment Successful! Image: ${IMAGE}:$BUILD_NUMBER"
    }
  }
}
