pipeline {
    agent any

    stages {
        // Declarative Jenkinsfile with a Docker-first Maven strategy and safe fallback
        pipeline {
            agent any

            stages {
                stage('Build') {
                    steps {
                        // Show Docker and environment info (non-fatal)
                        sh 'docker --version || true'
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
                                docker.image('maven:3.9.4-eclipse-temurin-11').inside('-v $HOME/.m2:/root/.m2') {
                                    sh 'mvn -B -V clean compile'
                                }
                            } else {
                                echo 'Docker not usable from this agent — falling back to local Maven or download'
                                // If Maven exists on the agent use it, otherwise download a temporary Maven and run it
                                def hasMvn = sh(script: 'which mvn >/dev/null 2>&1 && echo yes || echo no', returnStdout: true).trim() == 'yes'
                                if (hasMvn) {
                                    sh 'mvn -B -V clean compile'
                                } else {
                                    // Download a small Maven distribution to /tmp and run it (stateless)
                                    sh '''
                                        set -e
                                        MAVEN_VERSION=3.9.4
                                        ARCHIVE=apache-maven-${MAVEN_VERSION}-bin.tar.gz
                                        URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/${ARCHIVE}
                                        echo "Downloading Maven ${MAVEN_VERSION}..."
                                        curl -fsSL "$URL" -o /tmp/${ARCHIVE}
                                        tar -xzf /tmp/${ARCHIVE} -C /tmp
                                        /tmp/apache-maven-${MAVEN_VERSION}/bin/mvn -B -V clean compile
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
                                docker.image('maven:3.9.4-eclipse-temurin-11').inside('-v $HOME/.m2:/root/.m2') {
                                    sh 'mvn -B -V failsafe:integration-test failsafe:verify'
                                }
                            } else {
                                echo 'Docker unavailable — running integration goals with local/downloaded Maven'
                                def hasMvn = sh(script: 'which mvn >/dev/null 2>&1 && echo yes || echo no', returnStdout: true).trim() == 'yes'
                                if (hasMvn) {
                                    sh 'mvn -B -V failsafe:integration-test failsafe:verify'
                                } else {
                                    sh '''
                                        set -e
                                        MAVEN_VERSION=3.9.4
                                        ARCHIVE=apache-maven-${MAVEN_VERSION}-bin.tar.gz
                                        URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/${ARCHIVE}
                                        curl -fsSL "$URL" -o /tmp/${ARCHIVE}
                                        tar -xzf /tmp/${ARCHIVE} -C /tmp
                                        /tmp/apache-maven-${MAVEN_VERSION}/bin/mvn -B -V failsafe:integration-test failsafe:verify
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