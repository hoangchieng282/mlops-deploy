pipeline {
    agent {
        docker {
            // image 'ultralytics/yolov5:latest'
            image 'openshift/origin-cli'
        }
    }
    
    // agent any
    // environment {
    //     KUBECONFIG = "/var/jenkins_home/my-kubeconfig"
    // }

    stages {
        // stage('Install oc') {
        //     steps {
        //         sh 'curl -L https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz | tar xvz'
        //         sh 'mv ./oc /usr/local/bin/oc'
        //         sh 'mv ./kubectl /usr/local/bin/kubectl'
        //         sh 'oc version'
        //     }
        //     }
        stage('Test OpenShift Cluster Connection') {
            steps {
                script {
                   
                        withCredentials([
                            usernamePassword(
                                credentialsId: 'openshift-sa-token-int-vn',
                                usernameVariable: 'USERNAME',
                                passwordVariable: 'OPENSHIFT_TOKEN'
                            )
                        ]){
                            sh "oc login --token=${OPENSHIFT_TOKEN} --server=https://api.sandbox-m3.1530.p1.openshiftapps.com:6443"
                            sh "oc whoami"
                        }
                }
                
            }
        }
    }
}