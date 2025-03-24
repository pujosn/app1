pipeline {
    agent any

    environment {
        DOCKER_IMAGE      = "pujosn/app-jam"
        IMAGE_TAG         = "1.0"
        GKE_CLUSTER       = "cluster-brook"
        GCP_PROJECT       = "robin-454008"
        STAGING_NAMESPACE = "staging-ns"
        PROD_NAMESPACE    = "prod-ns"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'git@github.com:pujosn/app1.git'
            }
        }

        stage('Testing') {
            steps {
                sh 'echo "Running HTML Validation"'
                sh 'tidy -errors index.html || true' // validasi HTML
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }

            stage('Deploy to Staging') {
                steps {
                    script {
                        withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud container clusters get-credentials $GKE_CLUSTER --zone us-central1-c
                        kubectl config set-context --current --namespace=${STAGING_NAMESPACE}
                        kubectl apply -f staging-deployment.yaml
                        kubectl apply -f pvc-staging.yaml
                        '''
                    }
                }
            }
        }

        stage('Manual Approval for Production') {
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
                
            }
        }

        stage('Deploy to Production') {
            steps {
                sh '''
                gcloud container clusters get-credentials $GKE_CLUSTER --zone us-central1-c
                kubectl config set-context --current --namespace=${PROD_NAMESPACE}
                kubectl apply -f prod-deployment.yaml
                kubectl apply -f pvc-prod.yaml 
                kubectl apply -f ingress.yaml
                kubectl apply -f cert-prod.yaml
                '''
            }
        }
    }
}