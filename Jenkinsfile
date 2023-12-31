pipeline {
    agent any
    stages {
        stage('SCM') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        credentialsId: 'jenkins',  // Specify the SSH credential ID here
                        url: 'https://github.com/Wakeelabdul/node-todo-cicd.git'
                    ]]
                ])
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
            environment {
                GIT_REPO_NAME = "node-todo-cicd"
                GIT_USER_NAME = "Wakeelabdul"
            }
            steps {
                withCredentials([string(credentialsId: 'gitHub', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def currentBuildNumber = currentBuild.number
                        def dockerHubUsername = "wakeel532687"  // Replace with your Docker Hub username
                        def updatedImageName = "${dockerHubUsername}/node-app-test:${currentBuildNumber}"
                
                        // Update Deployment YAML
                        sh "sed -i 's|image:.*|image: ${updatedImageName}|' k8s/deployment.yaml"
                
                        // Update Pod YAML if necessary
                        sh "sed -i 's|image:.*|image: ${updatedImageName}|' k8s/pod.yaml"
                
                        // Commit the changes to GitHub
                        sh "git config --global user.email 'wakeelabdul50512@gmail.com'"
                        sh "git config --global user.name 'Mohammed Abdul Wakeel'"
                        sh "git add k8s/deployment.yaml k8s/pod.yaml"
                        sh "git commit -m 'Update image in Deployment and Pod'"
                        sh "git remote set-url origin https://${GITHUB_TOKEN}@https://github.com/Wakeelabdul/node-todo-cicd.git"
                        sh "git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master"
                    }
                }
            }
        }
    }
}
