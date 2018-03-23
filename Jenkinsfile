pipeline{
    agent any
    stages {
        stage('checkout code') {
            steps {
                deleteDir()
                echo "checkout code"
                sh 'touch somesamplecode.go'
                sh 'ls -la'
            }
        }
        stage('build something') {
            parallel {
                stage ('build centos') {
                    agent { docker image: 'centos:latest' }
                    steps {
                        sh 'ls -la'
                        sh "cat /etc/redhat-release"
                        sh "cat /etc/redhat-release > centos.txt"
                        stash includes: "*.txt", name: "centos-stash"

                    }
                }
                stage ('build ubuntu') {
                    agent { docker 'ubuntu:latest' }
                    steps {
                        sleep 20
                        sh 'ls -la'
                        sh "cat /etc/os-release"
                        sh "cat /etc/os-release > ubuntu.txt"
                        stash includes: "*.txt", name: "ubuntu-stash"
                    }
                }
            }
        }
        stage('upload artifact') {
            steps {
                sh "ls -la"
                unstash 'centos-stash'
                sh "ls -la"
                unstash 'ubuntu-stash'
                sh "ls -la"
                echo "would upload something to s3"
            }
        }
        stage('validation') {
            steps {
                slackSend (color: '#00FF00', channel: '#validations', message: "Waiting for input validation on whether the build worked @ (${env.RUN_DISPLAY_URL})")
                
                input message: "did the build work????", ok: "DUHH"
                echo "validated yay!!!"
            }
        }
        stage('integration test env setup') {
            parallel {
                stage ('wait for env1 creation') {
                    steps {
                        echo "waiting..."
                    }
                }
                stage ('wait for env2 creation') {
                    steps {
                        echo "waiting..."
                    }
                }
            }
        }
        stage ('run integration tests') {
            parallel {
                stage ('run tests in env1') {
                    steps {
                        echo "running env1 tests"
                    }
                }
                stage ('run tests in env2') {
                    steps {
                        echo "running env2 tests"
                    }
                }
            }
        }
        stage ('Validation') {
            steps {
                input message: "how do the tests look?", ok: "awesome!"
            }
        }
        stage('send notification to beta channel') {
            when {
                branch 'master'
            }
            steps {
                echo "would notify beta channel"
            }
        }
    }
    post {
        always {
            slackSend (color: '#00FF00', channel: '#general', message: "BUILD COMPLETED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.RUN_DISPLAY_URL})")
        }
    }
}
