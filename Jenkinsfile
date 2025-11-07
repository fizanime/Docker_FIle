//Scripted
/*node {
    stage('Build') {
        echo "Build"
    }   

    stage('Test') {
        echo "Test"
    }

    stage('Integration test') {
        echo "Integration test"
    }
}
*/
//Declarative
pipeline {
    agent any
    
    environment {
        dockerHome = tool 'docker'
        mavenHome = tool 'Maven3'
        PATH = "@dockerHome@/bin:@mavenHome@/bin:${env.PATH}"
    }
    
    stages {
        stage('Build') {
            steps {
                //sh 'mvn --version'
                //sh 'node --version'
                sh 'docker --version'
                echo "Build"
                echo "BUILD_NUMBER - ${env.BUILD_NUMBER}"
                echo "PATH - $PATH"
                echo "JOB_NAME - ${env.JOB_NAME} "
                echo "BUILD_ID - ${env.BUILD_ID} "
            }
        }
        stage('Test') {
            steps{
                echo "Test"
            }
        }
        stage('Integration test') {
            steps{
                echo "Integration test"
            }
        }
    }
    post {
        always {
            echo "This will always run"
        }
        success {
            echo "This will run only if successful"
        }
        failure {
            echo "This will run only if failed"
        }
    }
}