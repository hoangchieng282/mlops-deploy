pipeline {
    agent any
    stages {
        stage('Test connection to OpenShift cluster') {
            steps {
                withOpenShift(
                    serverUrl: 'https://api.sandbox-m3.1530.p1.openshiftapps.com:6443',
                    project: 'gramphoang-dev',
                    credentialsId: 'jenkins-sa-key'
                ) {
                    sh 'oc whoami'
                }
            }
        }
    }
}