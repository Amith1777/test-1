
pipeline {
    agent any

    tools {
        jdk 'java-11'
        maven 'maven'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Amith1777/test-1.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }

        stage('codescan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-jenkins', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=http://13.48.137.45:9000/"
                }
            }
        }

        stage('Build and tag') {
            steps {
                sh "docker build -t Amith1777/java-ci:1 ."
            }
        }

        stage('Docker image scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html Amith1777/java-ci:1"
            }
        }

        stage('Containersation') {
            steps {
                sh '''
                    docker stop c3 || true
                    docker rm c3 || true
                    docker run -it -d --name c3 -p 9004:8080 Amith1777/java-ci:1
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Jenkins-DockerHub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Pushing image to repository') {
            steps {
                sh 'docker push Amith1777/java-ci:1'
            }
        }
    }
}
