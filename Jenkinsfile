pipeline {
    agent {
        docker {
            // image 'ultralytics/yolov5:latest'
            image 'openshift/origin-cli'
        }
    }
    
    parameters {
        string(name: 'IMAGE_NAME', description: 'The image to be deployed')
        // string(name: 'VERSION', description: 'The version for the model')
    }
    // agent any
    environment {
        IMAGE_TO_DEPLOY="mlops-backend:${params.IMAGE_NAME}"
        DOCKER_REPO="mlops-docker-images"
    }


    stages {
        stage('Test OpenShift Cluster Connection') {
            steps {
                script {
                    try {
                        withCredentials([
                            usernamePassword(
                                credentialsId: 'okd-cluster-admin',
                                usernameVariable: 'USERNAME',
                                passwordVariable: 'SA_JENKINS_TOKEN'
                            )
                        ]){
                            sh "oc login --token=${SA_JENKINS_TOKEN} --server=https://api.sandbox-m3.1530.p1.openshiftapps.com:6443"
                            sh "oc whoami"
                        }
                    }catch {
                        
                    }
                }
            }
        }

        stage('Precheck params'){
            steps {
                sh 'echo Hello'
            }
        }

        stage('Start deploy new image version'){
            steps {
                script {
                    sh "oc set dc/backend-mlops backend-mlops=artifactorymlopsk18.jfrog.io/${DOCKER_REPO}/${IMAGE_TO_DEPLOY}"
                    sh "oc rollout status dc/backend-mlops"
                }
            }
        }
    }
}