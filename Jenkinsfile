pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Sanity check that Docker is available on the Jenkins node
                sh 'docker --version'
                echo "Build"
                echo "BUILD_NUMBER - ${env.BUILD_NUMBER}"
                echo "PATH - $PATH"
                echo "JOB_NAME - ${env.JOB_NAME}"
                echo "BUILD_ID - ${env.BUILD_ID}"
            }
        }
        stage('Compile') {
            agent {
                docker {
                    image 'maven:3.9.4-eclipse-temurin-11'
                    // mount the host Maven repo to speed up builds and reuse deps
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                // run Maven in batch/quiet mode and show versions
                sh 'mvn -B -V clean compile'
            }
        }

        stage('Test') {
            steps {
                echo 'Test'
            }
        }

        stage('Integration test') {
            agent {
                docker {
                    image 'maven:3.9.4-eclipse-temurin-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn -B -V failsafe:integration-test failsafe:verify'
                echo 'Integration test'
            }
        }
    }

    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
    }
}