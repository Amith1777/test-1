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
                sh "mvn compile"
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }

        stage('Code Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-jenkins', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.login=$SONAR_AUTH_TOKEN \
                        -Dsonar.host.url=http://16.16.233.229:9000/
                    """
                }
            }
        }

        stage('Build and Tag Image') {
            steps {
                sh "docker build -t Amith1777/java-ci-project-image:1 ."
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html Amith1777/java-ci-project-image:1"
            }
        }

        stage('Containerization') {
            steps {
                sh """
                    docker stop c1 || true
                    docker rm c1 || true
                    docker run -it -d --name c1 -p 9001:8080 Amith1777/java-ci-project-image:1
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Jenkins-DockerHub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                }
            }
        }

        stage('Push Image to Repository') {
            steps {
                sh "docker push Amith1777/java-ci-project-image:1"
            }
        }
    }
}
