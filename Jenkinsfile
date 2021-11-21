pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "brashed/train-schedule"
        DOCKER_HUB_CRED = credentials('DOKCER_REG_ACC')
    }
    stages {
        stage('Build') {
            steps {
                sh 'printenv'
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('', 'DOKCER_REG_ACC') {//withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'DOKCER_REG_ACC', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        sh "docker tag ${DOCKER_IMAGE_NAME}:latest ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                script {
                            sh """
                              export CANARY_REPLICAS=${DOCKER_IMAGE_NAME}
                              export DOCKER_IMAGE_NAME=${DOCKER_IMAGE_NAME}
                              export BUILD_NUMBER=${env.BUILD_NUMBER}
                              
                              gcloud auth activate-service-account --key-file=/home/edureka/Downloads/ingka-cff-slm-dev-923c78677045.json --project=ingka-cff-slm-dev
                              gcloud container clusters get-credentials slm-cluster --zone europe-west4-a --project ingka-cff-slm-dev

                              kubectl apply -f train-schedule-kube-canary.yml

                              """
                }      
            }
        }
        /*
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        */
    }
}
