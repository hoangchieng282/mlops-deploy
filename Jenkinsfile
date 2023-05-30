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
        choice(name: 'MODULE', choices: ['back','front'], description: 'Select the module to be deployed')
        choice(name: 'ENV', choices: ['test','prod'], description: 'Select the environment to deploy')
        // string(name: 'VERSION', description: 'The version for the model')
    }
    // agent any
    environment {
        //Basic image to be deploy
        def IMAGE_TO_DEPLOY="mlops-backend"

        //Artifactory connect info
        def DEPLOYMENTCONFIG="backend-mlops"
        def DOCKER_REPO="mlops-docker-images"
        //OPENSHIFT info
        def KUBECONFIG='/tmp/kubeconfig.yaml'
        def OKD_SERVER="${env.TESTING_OKD_ENV}"
        def OKD_CRED_KEY='okd-cluster-admin'
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
                    if(params.ENV == 'prod'){
                        OKD_SERVER="${env.PROD_OKD_ENV}"
                        OKD_CRED_KEY='okd-jenkins-token-prod'
                    }
                    
                    echo "Checking image version on Artifactory"
                    sh "curl -u ${USERNAME}:${PASSWORD} -f -I https://${env.SERVER_URL}/artifactory/${DOCKER_REPO}/${IMAGE_TO_DEPLOY}/${params.IMAGE_NAME}/manifest.json"
                    } 
                }
            }
        }
        stage('Test OpenShift Cluster Connection') {
            steps {
                script {
                        withCredentials([
                            usernamePassword(
                                credentialsId: "${OKD_CRED_KEY}",
                                usernameVariable: 'USERNAME',
                                passwordVariable: 'SA_JENKINS_TOKEN'
                            )
                        ]){
                            sh "oc login --token=${SA_JENKINS_TOKEN} --server=${OKD_SERVER}"
                            sh "oc whoami"
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

                    sh "oc set image dc/${DEPLOYMENTCONFIG} ${DEPLOYMENTCONFIG}=${env.SERVER_URL}/${DOCKER_REPO}/${IMAGE_TO_DEPLOY}:${params.IMAGE_NAME}"
                    sh "oc rollout status dc/${DEPLOYMENTCONFIG}"
                }
            }
        }
    }
    post {
        success {
            slackSend(color:"good", message:"To: <!here|here>, Build deployed successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            sh "curl https://mlops-models-server-1gp6.vercel.app/jenkinsResult/success"
        }

        failure {
            slackSend(color:"#ff0000",message: "To: <!channel|channel>, Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            sh "curl https://mlops-models-server-1gp6.vercel.app/jenkinsResult/fail"
        }
        
        always {
            cleanWs()
        }
    }
}