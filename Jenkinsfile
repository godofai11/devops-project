pipeline {
    agent any

    tools {
        maven 'Maven 3.9.9'
        jdk 'OpenJDK 11'
    }

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        ARTIFACTORY_URL = 'https://your-real-org.jfrog.io/artifactory' // Replace with your actual Artifactory URL
    }

    stages {
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
