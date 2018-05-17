library 'kypseli'
pipeline {
  options { 
    buildDiscarder(logRotator(numToKeepStr: '5')) 
  }
  agent {
    kubernetes {
      label 'maven'
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
                            //sh 'mvn verify'
                            echo "verify"
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
        stage('Docker Build and Push') {
          agent none
        //checkpoint 'Before Docker Build and Push'
          steps {
            script {
                def label = "kaniko-${UUID.randomUUID().toString()}"
                 podTemplate(name: 'kaniko', label: label, namespace:'kaniko', yaml: """
                 kind: Pod
                 metadata:
                   name: kaniko
                 spec:
                   containers:
                   - name: kaniko
                     image: beedemo/kaniko:jenkins-k8s-3 # we need a patched version of kaniko for now
                     workingDir: /home/root
                     imagePullPolicy: Always
                     command:
                     - cat
                     tty: true
                     volumeMounts:
                       - name: jenkins-docker-cfg
                         mountPath: /root/.docker
                   volumes:
                     - name: jenkins-docker-cfg
                       secret:
                         secretName: jenkins-docker-cfg
                         items:
                         - key: .dockerconfigjson
                           path: config.json
                 """
                   ) {
                   node(label) {
                     container('kaniko') {
                       unstash 'jar-dockerfile'
                       sh "cd target && /kaniko/executor -c . -v debug --destination=beedemo/mobile-deposit-api:kaniko-1"
                     }
                   }
                 }
            }
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

