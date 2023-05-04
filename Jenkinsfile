pipeline {
    agent any
    
    stages {
        stage('Test OpenShift Cluster Connection') {
            steps {
                script {
                    openshift.withCluster() {
                        sh "oc project"
                    }
                }
                
            }
        }
    }
}