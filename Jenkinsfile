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
    agent any
    stages {
        stage('Build') {
            steps {
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
}