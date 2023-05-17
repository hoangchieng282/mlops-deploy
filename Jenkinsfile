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
        def IMAGE_TO_DEPLOY="mlops-backend"

        //Artifactory connect info
        def DEPLOYMENTCONFIG="backend-mlops"
        def DOCKER_REPO="mlops-docker-images"
        def SERVER_URL="artifactorymlopsk18.jfrog.io"
        //OPENSHIFT info
        def KUBECONFIG='/tmp/kubeconfig.yaml'
        def OKD_SERVER="https://api.sandbox-m3.1530.p1.openshiftapps.com:6443"

    }


    stages {
        stage('Precheck params'){
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'artifactory-chih',
                            usernameVariable: 'USERNAME',
                            passwordVariable: 'PASSWORD'
                        )
                    ]){
                    echo "Validating parameters..."
                    if (!params.IMAGE_NAME?.trim()) {
                        error "IMAGE_NAME is a mandatory parameter"
                        return
                    }
                    if(params.MODULE == 'front'){
                        IMAGE_TO_DEPLOY="mlops-frontend"
                        DEPLOYMENTCONFIG="frontend-mlops"
                    }
                    
                    echo "Checking image version on Artifactory"
                    sh "curl -u ${USERNAME}:${PASSWORD} -f -I https://${SERVER_URL}/artifactory/${DOCKER_REPO}/${IMAGE_TO_DEPLOY}/${params.IMAGE_NAME}/manifest.json"
                    } 
                }
            }
        }
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

        stage('Start deploy new image version'){
            steps {
                script {
                    sh "echo ${DEPLOYMENTCONFIG}"
                    def currentReplicas = sh (
                        script: "oc get dc/${DEPLOYMENTCONFIG} -o jsonpath='{.spec.replicas}'",
                        returnStdout: true
                    ).trim()

                    // Scale the deployment to 1 replica if the current count is 0
                    if (currentReplicas == '0') {
                        sh "oc scale --replicas=1 dc/${DEPLOYMENTCONFIG}"
                    }

                    sh "oc set image dc/${DEPLOYMENTCONFIG} ${DEPLOYMENTCONFIG}=artifactorymlopsk18.jfrog.io/${DOCKER_REPO}/${IMAGE_TO_DEPLOY}:${params.IMAGE_NAME}"
                    sh "oc rollout status dc/${DEPLOYMENTCONFIG}"
                }
            }
        }
    }
    post {
        success {
            slackSend(color:"good", message:"To: <!here|here>, Build deployed successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        failure {
            slackSend(color:"#ff0000",message: "To: <!channel|channel>, Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        
        always {
            cleanWs()
        }
    }
}