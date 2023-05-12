#!/usr/bin/env groovy

pipeline {
    agent {
        docker {
            // image 'ultralytics/yolov5:latest'
            image 'openshift/origin-cli'
        }
    }
    
    parameters {
        string(name: 'IMAGE_NAME', description: 'The image to be deployed')
        choice(name: 'MODULE', choices: ['front', 'back'], description: 'Select the module to be deployed')
        // string(name: 'VERSION', description: 'The version for the model')
    }
    // agent any
    environment {
        //Basic image to be deploy
        def IMAGE_TO_DEPLOY="mlops-backend:${params.IMAGE_NAME}"

        //Artifactory connect info
        def DEPLOYMENTCONFIG="backend-mlops"
        def DOCKER_REPO="mlops-docker-images"
        //OPENSHIFT info
        def KUBECONFIG='/tmp/kubeconfig.yaml'
        def OKD_SERVER="https://api.sandbox-m3.1530.p1.openshiftapps.com:6443"
        if(params.MODULE == 'front'){
            IMAGE_TO_DEPLOY="mlops-frontend:${params.IMAGE_NAME}"
            DEPLOYMENTCONFIG="frontend-mlops"
        }
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
                            sh "oc login --token=${SA_JENKINS_TOKEN} --server=${OKD_SERVER}"
                            sh "oc whoami"
                        }
                    }catch(e) {
                        
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
                    sh "oc set image dc/${DEPLOYMENTCONFIG} ${DEPLOYMENTCONFIG}=artifactorymlopsk18.jfrog.io/${DOCKER_REPO}/${IMAGE_TO_DEPLOY}"
                    sh "oc rollout status dc/${DEPLOYMENTCONFIG}"
                }
            }
        }
    }
}