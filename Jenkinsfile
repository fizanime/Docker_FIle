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
}*/

//Declarative
pipeline {
    agent {docker{image 'maven:3.8.1' }}
    stages {
        stage('Build') {
            steps {
                sh 'mvn --version'
                echo "Build"
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