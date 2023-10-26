pipeline {
    agent any
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
                script {
                    def currentBuildNumber = currentBuild.number
                    sh "docker build . -t node-app-test-${currentBuildNumber}"
                }
            }
        }
        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    script {
                        def currentBuildNumber = currentBuild.number
                        sh "docker tag node-app-test-${currentBuildNumber} ${env.dockerHubUser}/node-app-test:${currentBuildNumber}"
                        sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                        sh "docker push ${env.dockerHubUser}/node-app-test:${currentBuildNumber}"
                    }
                }
            }
        }
        stage("Update Deployment and Pod") {
            steps {
            script {
                def currentBuildNumber = currentBuild.number
                def updatedImageName = "${env.dockerHubUser}/node-app-test:${currentBuildNumber}"
            
                // Update Deployment YAML
                sh "sed -i 's|image:.*|image: ${updatedImageName}|' your-deployment.yaml"
            
                // Update Pod YAML if necessary
                // sh "sed -i 's|image:.*|image: ${updatedImageName}|' your-pod.yaml"
            
                // Commit the changes to GitHub
                sh "git add k8s/deployment.yaml k8s/pod.yaml"
                sh "git commit -m 'Update image in Deployment and Pod'"
                sh "git push origin master"  // You can replace 'master' with your branch name
                }
            }
        }
    }
}
