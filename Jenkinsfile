library 'kypseli'
pipeline {
    options { 
        buildDiscarder(logRotator(numToKeepStr: '5')) 
    }
    agent none
    stages {
        stage('Prep') {
            steps {
                gitShortCommit(7)
            }
        }
        stage('Build') {
            steps {
                mavenCacheBuild("mobile-deposit-api")
            }
        }
    }
}

