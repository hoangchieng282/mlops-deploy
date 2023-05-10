pipeline {
    agent {
        docker {
            // image 'ultralytics/yolov5:latest'
            image 'openshift/origin-cli'
        }
    }

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
                   
                        openshift.withCluster('mlops-cluster') {

                            // Use the OpenShift client
                            sh "oc whoami"
                        
                        }
                }
                
            }
        }
    }
}