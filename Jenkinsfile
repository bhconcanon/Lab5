@Library("startendtime") _
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: build
    image: 'maven:3.8.3-jdk-11'
    command:
    - cat
    tty: true
    '''
        defaultContainer 'build'
        }
    }
    
    triggers {
        eventTrigger jmespathQuery("lab=='5'")
    }

    stages {
        stage ('buildStart Time Stage') {
            steps {
                buildStart ()
            }
        }
        stage ('build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test') {
           when { 
            expression {
                BRANCH_NAME == 'main' || BRANCH_NAME == 'release'
                } 
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Test Optional') {
           when {
                triggeredBy 'test=enabled'
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Stop Dev Deploy') {
           when {
            not {
                branch 'develop'
                }
            }
            steps {
                sh './scripts/deliver.sh'
            }
        }

        stage('Deploy') { 
                when {
                    triggeredBy 'deploy=enabled'
            }
            steps {
                sh './scripts/deliver.sh'
                }
            }

        stage('buildEnd Time Stage') {
            steps {
                buildEnd ()
                }
            }
        }

        post {
            success {
                emailext (
                    subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: """SUCCESSFUL: Job '${JOB_NAME} [${BUILD_NUMBER}]':
                    Check console output at ${BUILD_URL}""",
                    to: 'bilal.hussain@concanon.com'
                )
            }
            failure {
                emailext (
                    subject: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: """FAULURE: Job '${JOB_NAME} [${BUILD_NUMBER}]':
                    Check console output at ${BUILD_URL}""",
                    to: 'bilal.hussain@concanon.com'
            )
        }    
    }
}