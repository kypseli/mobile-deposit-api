script {
    def buildVersion = null
    def short_commit = null
}

pipeline {
    options { buildDiscarder(logRotator(numToKeepStr: '5')) }
    agent { docker 'kmadel/maven:3.3.3-jdk-8' }
    stages {
        stage('Example Build') {
            steps {
                script {
                    git_commit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    short_commit=git_commit.take(7)
                    echo short_commit
                    env.SHORT_COMMIT = short_commit
                }
                sh 'mvn -DGIT_COMMIT="${SHORT_COMMIT}" -DBUILD_NUMBER=${BUILD_NUMBER} -DBUILD_URL=${BUILD_URL} clean package'
            }
        }
        stage('Quality Analysis') {
            when {
                expression { !env.BRANCH_NAME.startsWith("PR") }
            }
            steps {
                parallel (
                    "integrationTests" : {
                        agent { docker 'kmadel/maven:3.3.3-jdk-8' }
                        sh 'mvn -Dmaven.repo.local=/data/mvn/repo verify'
                        
                    },
                    "sonarAnalysis" : {
                        agent { docker 'kmadel/maven:3.3.3-jdk-8' }
            			environment {
            				SONAR = credentials('sonar.beedemo')
            			}
                        sh 'mvn -Dmaven.repo.local=/data/mvn/repo -Dsonar.scm.disabled=True -Dsonar.login=$SONAR sonar:sonar'
                    }    
                )
            }
            
        }
    }
    post {
        success {
            hipchatSend color: 'GREEN', message: "${env.JOB_NAME} ${env.BUILD_NUMBER} status: SUCCESS <a href=\'${env.BUILD_URL}\'>Open</a>", room: '1613593', server: 'cloudbees.hipchat.com', credentialId: 'hipchat-sa-demo-environment', v2enabled: true
        }
        failure {
            hipchatSend color: 'RED', message: "${env.JOB_NAME} ${env.BUILD_NUMBER} status: FAILURE <a href=\'${env.BUILD_URL}\'>Open</a>", room: '1613593', server: 'cloudbees.hipchat.com', credentialId: 'hipchat-sa-demo-environment', v2enabled: true
        }
    }
}