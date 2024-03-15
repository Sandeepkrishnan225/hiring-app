pipeline {
    agent any

        environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        
       
        stage('Docker Build') {
            steps {
                sh "docker build . -t sandeepkrishnan225/hiring-app:$BUILD_NUMBER"
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub', variable: 'hubPwd')]) {
                    sh "docker login -u sandeepkrishnan225 -p ${hubPwd}"
                    sh "docker push sandeepkrishnan225/hiring-app:$BUILD_NUMBER"
                }
            }
        }
        stage('Checkout K8S manifest SCM'){
            steps {
              git branch: 'main', url: 'https://github.com/Sandeepkrishnan225/Hiring-app-argocd.git'
            }
        } 
        stage('GitOps Enabler to update deployment manifest') {
            environment {
                GIT_REPO_NAME = "Hiring-app-argocd"
                GIT_USER_NAME = "Sandeepkrishnan225"
                GIT_USER_EMAIL = "perugusandeep25@gmail.com"  
        }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    script {

                // Specify the target directory for the clone
                        def cloneDirectory = "hiringapp-build-push_update_maifest"

                // Remove the existing directory if it exists
                        sh "rm -rf ${cloneDirectory}"

                // Clone the repository
                        sh "git clone https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} ${cloneDirectory}"
                        dir("${cloneDirectory}/dev") {
                    // Fetch the current Deployment YAML
                            sh "wget https://raw.githubusercontent.com/${GIT_USER_NAME}/${GIT_REPO_NAME}/main/dev/deployment.yaml"

                    // Modify the image tag in the Deployment YAML
                            sh "sed -i 's|replicas:.*/replicas: ${BUILD_NUMBER}|g' /var/lib/jenkins/workspace/hiringapp-build-push_update_maifest/hiringapp-build-push_update_maifest/dev/deployment.yaml"
                
                    // Add, commit, and push the changes
                            sh "git add ."
                            sh "git commit -m 'Updated deployment image to version ${BUILD_NUMBER}'"
                            sh "git push origin main"
                }
            }
        }
    }
    }
            }
} 
