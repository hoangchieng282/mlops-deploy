pipeline {
    agent any
    
    stages {
        stage('Test OpenShift Cluster Connection') {
            steps {
                openshift.withCluster() {
                    sh "oc project"
                }
            }
        }
    }
}