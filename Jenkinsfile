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
    // GKE Regional clusters use region, not zone.
    // If you need a specific zone for node operations, you can declare it separately.
    GCP_SA_KEY = credentials('gcp-sa-key')  // secret file
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Dhiraj-Shivade/Jenkins-Trivy-SBOM-GKE-CI-CD.git'
      }
    }

    stage('Auth to GCP') {
      steps {
        // Use withCredentials to handle the service account key securely
        withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'SA_KEY_FILE')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$SA_KEY_FILE
            gcloud config set project ${PROJECT_ID}
            gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
          '''
        }
      }
    }

    stage('Install Docker CLI (if needed)') {
      steps {
        // This is not needed if your slave VM has Docker pre-installed.
        // It's not recommended for production.
        // The `docker.io` package name is for apt-based systems.
        sh '''
          if ! command -v docker >/dev/null 2>&1; then
            sudo apt-get update
            sudo apt-get install -y docker.io
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
        // Run Trivy scan in a Docker container with correct volume mapping
        sh '''
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --format table ${IMAGE}:$BUILD_NUMBER
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --format cyclonedx --output sbom.cdx ${IMAGE}:$BUILD_NUMBER
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE}:$BUILD_NUMBER
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
        // Use `get-credentials` with the region for regional clusters
        sh '''
          gcloud container clusters get-credentials ${CLUSTER} --region ${REGION} --project ${PROJECT_ID}
          
          # Apply service first
          kubectl apply -f k8s/service-clusterip.yaml
          
          # Use `kubectl set image` for a more reliable update
          kubectl set image deployment/python-app-deploy python-app-container=${IMAGE}:${BUILD_NUMBER} --record
          
          # wait for rollout
          kubectl -n default rollout status deployment/python-app-deploy --timeout=120s
        '''
      }
    }

    stage('Smoke Test (optional)') {
      steps {
        sh '''
          # Wait for deployment to stabilize before attempting to test
          kubectl wait --for=condition=available deployment/python-app-deploy --timeout=180s
          
          POD_NAME=$(kubectl get pods -l app=python-app -o jsonpath="{.items[0].metadata.name}")
          kubectl port-forward $POD_NAME 8080:5000 & FPID=$!
          sleep 5
          
          # Retry curl to handle temporary connection issues
          for i in {1..5}; do
            curl_output=$(curl -sS http://localhost:8080/)
            if [ $? -eq 0 ]; then
              echo "Smoke test successful: $curl_output"
              break
            fi
            echo "Smoke test failed on attempt $i, retrying..."
            sleep 2
          done
          
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
