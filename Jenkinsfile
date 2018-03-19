pipeline {
    options { 
        buildDiscarder(logRotator(numToKeepStr: '5')) 
    }
    agent {
        label 'docker'
    }
    environment {
        DOCKER_HUB_USER = 'beedemo'
        DOCKER_CREDENTIAL_ID = 'docker-hub-beedemo'
    }
    stages {
        stage('Prep') {
            steps {
                gitShortCommit(7)
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    sh "mvn -DGIT_COMMIT='${SHORT_COMMIT}' -DBUILD_NUMBER=${BUILD_NUMBER} -DBUILD_URL=${BUILD_URL} clean package"
                }
                junit allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml'
                stash name: 'jar-dockerfile', includes: '**/target/*.jar,**/target/Dockerfile'
            }
        }
        stage('Quality Analysis') {
            failFast true
            parallel {
                stage('Integration Tests') {
                    steps {
                        container('maven') {
                            sh 'mvn verify'
                        }
                    }
                }
                stage('Sonar Analysis') {
                    environment {
                        SONAR = credentials('sonar.beedemo')
                    }
                    steps {
                        withSonarQubeEnv('beedemo') {
                            container('maven') {
                                sh 'mvn -Dsonar.scm.disabled=True -Dsonar.login=$SONAR -Dsonar.branch=$BRANCH_NAME sonar:sonar'
                            }
                        }
                    }
                }
            }
        }
        stage('Quality Gate') {
            agent none
            steps {
                timeout(time: 4, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline failure due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage('Build & Push Docker Image') {
            environment {
                DOCKER_TAG = "${BUILD_NUMBER}-${SHORT_COMMIT}"
            }
            when {
                beforeAgent true
                branch 'master'
            }
            steps {
                checkpoint 'Before Docker Build and Push'
                unstash 'jar-dockerfile'
                container('docker') {
                    dockerBuildPush("${DOCKER_HUB_USER}", "mobile-deposit-api", "${DOCKER_TAG}", "target", "${DOCKER_CREDENTIAL_ID}")
                }
            }
        }
        stage('Deploy') {
            environment {
                DOCKER_TAG = "${BUILD_NUMBER}-${SHORT_COMMIT}"
            }
            when {
                branch 'master'
            }
            input {
                message "Deploy to prod?"
                ok "Yes"
                submitter "kypseli*ops"
            }
            agent none
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    checkpoint 'Before Deploy'
                    kubeDeploy('mobile-deposit-api', 'beedemo', "${DOCKER_TAG}", "prod")
                }
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

