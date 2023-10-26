pipeline {
    agent any
    environment {
        // Define an initial version number
        IMAGE_VERSION = currentBuild.number
    }
    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage("Clone Code") {
            steps {
                git url: "https://github.com/Wakeelabdul/node-todo-cicd.git", branch: "master"
            }
        }

        stage("Build and Test") {
            steps {
                // Use the current build number as the version number
                sh "docker build . -t node-app-test-${env.IMAGE_VERSION}"
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    // Use the current build number as the version number in the image tag
                    sh "docker tag node-app-test-${env.IMAGE_VERSION} ${env.dockerHubUser}/node-app-test:${env.IMAGE_VERSION}"
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker push ${env.dockerHubUser}/node-app-test:${env.IMAGE_VERSION}"
                }
            }
        }
    }
}
