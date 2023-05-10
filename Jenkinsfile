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
                    try {
                        withCredentials([
                            usernamePassword(
                                credentialsId: 'okd-cluster-admin',
                                usernameVariable: 'USERNAME',
                                passwordVariable: 'PASSWORD'
                            )
                        ]){
                            sh "oc login -u=${USERNAME} -p=${PASSWORD} --server=https://api.sandbox-m3.1530.p1.openshiftapps.com:6443"
                            sh "oc whoami"
                        }
                    }catch {

                    }
                }
                
            }
        }
    }
}