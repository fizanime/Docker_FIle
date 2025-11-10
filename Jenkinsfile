// Declarative Jenkinsfile with reliable Maven configuration
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    // Create Maven settings with mirrors and retry config
                    writeFile file: 'settings.xml', text: '''<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <mirrors>
    <mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>http://0.0.0.0/</url>
      <blocked>true</blocked>
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>retry</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <maven.wagon.http.retryHandler.count>3</maven.wagon.http.retryHandler.count>
        <maven.wagon.http.retryHandler.requestSentEnabled>true</maven.wagon.http.retryHandler.requestSentEnabled>
        <maven.wagon.http.pool>false</maven.wagon.http.pool>
        <maven.wagon.httpconnectionManager.maxPerRoute>2</maven.wagon.httpconnectionManager.maxPerRoute>
      </properties>
    </profile>
  </profiles>
</settings>'''
                }
                echo "Build"
                echo "BUILD_NUMBER - ${env.BUILD_NUMBER}"
                echo "PATH - $PATH"
                echo "JOB_NAME - ${env.JOB_NAME}"
                echo "BUILD_ID - ${env.BUILD_ID}"
            }
        }

        stage('Compile') {
            steps {
                script {
                    // Try to use Docker image if the Docker daemon is reachable.
                    def canUseDocker = sh(script: 'docker info >/dev/null 2>&1 && echo yes || echo no', returnStdout: true).trim() == 'yes'

                    if (canUseDocker) {
                        echo 'Using Maven Docker image for compile'
                        // Use the Docker Pipeline helper to run the image if plugin is available
                        docker.image('maven:3.9.5-eclipse-temurin-17').inside('-v $HOME/.m2:/root/.m2') {
                            sh 'mvn -B -V -s settings.xml --fail-fast clean compile'
                        }
                    } else {
                        echo 'Docker not usable from this agent — falling back to local Maven or download'
                        // If Maven exists on the agent use it, otherwise download a temporary Maven and run it
                        def hasMvn = sh(script: 'which mvn >/dev/null 2>&1 && echo yes || echo no', returnStdout: true).trim() == 'yes'
                        if (hasMvn) {
                            sh 'mvn -B -V -s settings.xml --fail-fast clean compile'
                        } else {
                            // Download a small Maven distribution to /tmp and run it (stateless)
                            sh '''
set -e
MAVEN_VERSION=3.9.5
ARCHIVE=apache-maven-${MAVEN_VERSION}-bin.tar.gz
URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/${ARCHIVE}
echo "Downloading Maven ${MAVEN_VERSION}..."
curl -fsSL "$URL" -o /tmp/${ARCHIVE}
tar -xzf /tmp/${ARCHIVE} -C /tmp
/tmp/apache-maven-${MAVEN_VERSION}/bin/mvn -B -V -s settings.xml --fail-fast clean compile
'''
                        }
                    }
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Test'
            }
        }

        stage('Integration test') {
            steps {
                script {
                    def canUseDocker = sh(script: 'docker info >/dev/null 2>&1 && echo yes || echo no', returnStdout: true).trim() == 'yes'
                    if (canUseDocker) {
                        echo 'Using Maven Docker image for integration tests'
                        docker.image('maven:3.9.5-eclipse-temurin-17').inside('-v $HOME/.m2:/root/.m2') {
                            sh 'mvn -B -V -s settings.xml --fail-fast failsafe:integration-test failsafe:verify'
                        }
                    } else {
                        echo 'Docker unavailable — running integration goals with local/downloaded Maven'
                        def hasMvn = sh(script: 'which mvn >/dev/null 2>&1 && echo yes || echo no', returnStdout: true).trim() == 'yes'
                        if (hasMvn) {
                            sh 'mvn -B -V -s settings.xml --fail-fast failsafe:integration-test failsafe:verify'
                        } else {
                            sh '''
set -e
MAVEN_VERSION=3.9.5
ARCHIVE=apache-maven-${MAVEN_VERSION}-bin.tar.gz
URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/${ARCHIVE}
curl -fsSL "$URL" -o /tmp/${ARCHIVE}
tar -xzf /tmp/${ARCHIVE} -C /tmp
/tmp/apache-maven-${MAVEN_VERSION}/bin/mvn -B -V -s settings.xml --fail-fast failsafe:integration-test failsafe:verify
'''
                        }
                    }
                }
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