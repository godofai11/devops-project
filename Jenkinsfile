pipeline {
    agent any

    tools {
        maven 'Maven 3.9.9'
        jdk 'OpenJDK 11'  // Ensure OpenJDK 11 is selected for this pipeline
    }

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        ARTIFACTORY_URL = 'https://your-real-org.jfrog.io/artifactory'
        JAVA_HOME = tool 'OpenJDK 11' // Explicitly set JAVA_HOME to OpenJDK 11
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}" // Ensure Java 11 is first in the PATH
    }

    stages {
        stage('Set Up Environment') {
            steps {
                script {
                    // Ensure JAVA_HOME and PATH are correctly set
                    env.JAVA_HOME = tool name: 'OpenJDK 11', type: 'jdk'
                    env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
                }
            }
        }

        stage('Debug Environment') {
            steps {
                sh 'echo "JAVA_HOME=$JAVA_HOME"'
                sh 'echo "PATH=$PATH"'
                sh 'which java || echo "Java not found!"'
                sh 'java -version || echo "java -version failed"'
                sh 'mvn -version || echo "mvn -version failed"'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    def dockerImage = docker.build("${ARTIFACTORY_URL}/docker-local/myapp:${BUILD_NUMBER}")
                    docker.withRegistry("${ARTIFACTORY_URL}", 'jfrog-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
