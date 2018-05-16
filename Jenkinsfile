library 'kypseli'
pipeline {
    options { 
        buildDiscarder(logRotator(numToKeepStr: '5')) 
    }
  agent {
    kubernetes {
      label 'mypod'
      defaultContainer 'jnlp'
      yaml """
      apiVersion: v1
      kind: Pod
      metadata:
        labels:
          some-label: some-label-value
      spec:
        containers:
        - name: maven
          image: maven:alpine
          command:
          - cat
          tty: true
          volumeMounts:
            - name: efs-pvc
              mountPath: "/root/.m2/repository"
        restartPolicy: "Never"
        volumes:
          - name: efs-pvc
            persistentVolumeClaim:
              claimName: mobile-deposit-api-test
      """
      }
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
                        //TODO
                        echo "TODO"
                    }
                }
            }
        }
        stage('Quality Gate') {
            options {
                timeout(time: 10, unit: 'MINUTES') 
            }
            agent none
            steps {
                echo "TODO"
                /*script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline failure due to quality gate failure: ${qg.status}"
                    }
                }*/
            }
        }
        stage('Build & Push Docker Image') {
            environment {
                DOCKER_TAG = "${BUILD_NUMBER}-${SHORT_COMMIT}"
            }

            steps {
                //checkpoint 'Before Docker Build and Push'
                //unstash 'jar-dockerfile'
                dir 'target'
                dockerBuildPush("mobile-deposit-api", "${DOCKER_TAG}")
            }
        }
        stage('Deploy') {
            options {
                timeout(time: 10, unit: 'MINUTES') 
            }
            environment {
                DOCKER_TAG = "${BUILD_NUMBER}-${SHORT_COMMIT}"
            }
            when {
                beforeAgent true
                branch 'master'
            }
            input {
                message "Deploy to prod?"
                ok "Yes"
                submitter "kypseli*ops"
            }
            steps {
                echo "TODO"
                //kubeDeploy('mobile-deposit-api', 'beedemo', "${DOCKER_TAG}", "prod")
            }
        }
    }
    post {
        success {
            echo "TODO send slack message"
        }
        failure {
            echo "TODO send slack message"
        }
    }
}

