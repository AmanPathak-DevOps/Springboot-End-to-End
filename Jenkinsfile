pipeline {
    agent {
        docker {
            image 'abhishekf5/maven-abhishek-docker-agent:v1'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                sh 'echo passed' 
                // git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/Springboot-End-to-End.git'
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://3.89.227.188:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }
        stage('Upload Code Artifacts') {
            steps {
                server = Artifactory.server 'frog-artifactory-server'
                def uploadSpec = """{
                    "files": [
                    {
                    "pattern": "target/*.jar",
                    "target": "libs-release-local/"
                    }
                ]}"""
                server.upload(uploadSpec)
            }
        }
        stage('Build & Push Docker Image') {
            environment {
                DOCKER_IMAGE = "avian19/spring-docker:${BUILD_NUMBER}"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Updating Deployment File') {
            environment {
                GIT_REPO_NAME = "Springboot-end-to-end"
                GIT_USER_NAME = "AmanPathak-DevOps"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "aman07pathak@gmail.com"
                        git config user.name "AmanPathak-DevOps"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        ls
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml
                        git add deployment.yml
                        git commit -m "Update deployment Image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                    '''
                }
            }
        }
    }
}