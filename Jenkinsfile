pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "sivaprasadpappala"
        IMAGE_NAME = "springboot-sample"
        KUBECONFIG_CREDENTIAL = "kubeconfig-minikube"  // Jenkins credential ID
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/sivaprasadpappala/springboot-sample.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                        passwordVariable: 'DOCKER_PASS',
                        usernameVariable: 'DOCKER_USER')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL}", variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG apply -f k8s/deployment.yaml
                        kubectl --kubeconfig=$KUBECONFIG apply -f k8s/service.yaml
                    """
                }
            }
        }
    }
}

