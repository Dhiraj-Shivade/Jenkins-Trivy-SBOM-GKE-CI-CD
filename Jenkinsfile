pipeline {
  agent {
    docker {
      image 'gcr.io/google.com/cloudsdktool/cloud-sdk:latest'
      args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    PROJECT_ID = "clear-booking-470907-q6"
    REGION = "asia-south1"
    REPO_NAME = "python-app-repo"
    IMAGE = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/python-app"
    CLUSTER = "cluster-ci"
    ZONE = "asia-south1-a"
    GCP_SA_KEY = credentials('gke-sa-key')  // secret text or file
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: ''
      }
    }

    stage('Auth to GCP') {
      steps {
        sh '''
          echo "$GCP_SA_KEY" > sa.json
          gcloud auth activate-service-account --key-file=sa.json
          gcloud config set project ${PROJECT_ID}
          gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
        '''
      }
    }

    stage('Install Docker CLI (if needed)') {
      steps {
        sh '''
          # If docker not present in image, install basic docker client (Debian/Ubuntu)
          if ! command -v docker >/dev/null 2>&1; then
            apt-get update && apt-get install -y docker.io
          fi
          docker --version
        '''
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t ${IMAGE}:$BUILD_NUMBER .'
        sh 'docker tag ${IMAGE}:$BUILD_NUMBER ${IMAGE}:latest'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh '''
          # Pull Trivy image and run scan. Allow non-zero exit only on high/critical.
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --format table ${IMAGE}:$BUILD_NUMBER || RC=$?; \
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --format cyclonedx --output sbom.cdx ${IMAGE}:$BUILD_NUMBER || true; \
          # fail build if high/critical found
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE}:$BUILD_NUMBER || exit 1
        '''
      }
      post {
        always {
          archiveArtifacts artifacts: 'sbom.cdx', fingerprint: true
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
          gcloud container clusters get-credentials ${CLUSTER} --zone ${ZONE} --project ${PROJECT_ID}
          # substitute image name in deployment manifest and apply
          sed -e "s|IMAGE_PLACEHOLDER|${IMAGE}:$BUILD_NUMBER|g" k8s/deployment.yaml | kubectl apply -f -
          # apply service (ClusterIP) - preferred for private access
          kubectl apply -f k8s/service-clusterip.yaml
          # if you want ILB instead, uncomment:
          # kubectl apply -f k8s/service-ilb.yaml
          # wait rollout
          kubectl -n default rollout status deployment/python-app-deploy --timeout=120s
        '''
      }
    }

    stage('Smoke Test (optional)') {
      steps {
        sh '''
          # If using ClusterIP, run a temporary port-forwarded curl to verify
          POD=$(kubectl get pods -l app=python-app -o jsonpath="{.items[0].metadata.name}")
          kubectl port-forward $POD 8080:5000 & FPID=$!
          sleep 2
          curl -sS http://localhost:8080/ || true
          kill $FPID || true
        '''
      }
    }
  }

  post {
    failure {
      echo "Pipeline failed. Check logs & Trivy report."
    }
    success {
      echo "Pipeline succeeded. Image: ${IMAGE}:$BUILD_NUMBER"
    }
  }
}
