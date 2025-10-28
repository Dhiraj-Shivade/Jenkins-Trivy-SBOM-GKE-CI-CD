pipeline {
    agent any

    environment {
        PROJECT_ID = "clear-booking-470907-q6"
        REGION = "asia-south1"
        REPO = "python-app-repo"
        IMAGE = "python-app"
        IMAGE_NAME = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${BUILD_NUMBER}"
        K8S_DIR = "k8s"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                trivy image --exit-code 0 --format table ${IMAGE_NAME} > trivy-report.txt
                trivy image --exit-code 1 ${IMAGE_NAME} || true
                trivy image --scanners vuln,secret --format cyclonedx --output sbom.json ${IMAGE_NAME}
                """
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt, sbom.json', fingerprint: true
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet"
                sh "docker push ${IMAGE_NAME}"
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh """
                gcloud container clusters get-credentials cluster-ci \
                    --region ${REGION} --project ${PROJECT_ID}

                kubectl apply -f ${K8S_DIR}/deployment.yaml
                kubectl apply -f ${K8S_DIR}/service-clusterip.yaml

                kubectl set image deployment/python-app-deploy \
                    python-app-container=${IMAGE_NAME}

                kubectl rollout status deployment/python-app-deploy
                """
            }
        }
    }

    post {
        failure {
            echo "❌ Build Failed — Check Trivy report"
        }
    }
}
