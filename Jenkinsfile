pipeline {
    agent any

    tools {
        jdk 'java-11'
        maven 'maven'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Amith1777/test-1.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Code Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-jenkins', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.login=$SONAR_AUTH_TOKEN \
                        -Dsonar.host.url=http://13.48.137.45:9000/
                    '''
                }
            }
        }

        stage('Build and Tag Image') {
            steps {
                sh 'docker build -t amith2774/java-ci:1 .'
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html amith2774/java-ci:1'
            }
        }

        stage('Containerization') {
            steps {
                sh '''
                    docker stop c3 || true
                    docker rm c3 || true
                    docker run -it -d --name c3 -p 9004:8080 amith2774/java-ci:1
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Jenkins-DockerHub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to Repository') {
            steps {
                sh 'docker push amith2774/java-ci:1'
            }
        }
    }
}
