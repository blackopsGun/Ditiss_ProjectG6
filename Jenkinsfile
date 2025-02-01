pipeline {
    agent any

    environment {
        NODE_HOME = '/usr/local/bin/node'
        SONAR_SERVER = 'http://192.168.75.132:9000'
        DOCKER_HUB_USERNAME = 'blackopsgun'
        IMAGE_NAME = 'moviedb'
        IMAGE_TAG = 'latest'
        K8S_NAMESPACE = 'default'
        ARGOCD_APP_NAME = 'moviedb-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/blackopsGun/Ditiss_ProjectG6.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Static Code Analysis - SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'Sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'sonar-scanner -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.sources=./ -Dsonar.host.url=$SONAR_SERVER'
                }
            }
        }

        stage('Build and Test') {
            steps {
                sh 'npm run build'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t $DOCKER_HUB_USERNAME/$IMAGE_NAME:$IMAGE_TAG .
                    """
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_HUB_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        """
                    }
                }
            }
        }
