pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.9'
    }
    
    environment {
        // JFrog Artifactory configuration
        ARTIFACTORY_URL = 'https://trialyowuby.jfrog.io'
        ARTIFACTORY_CREDENTIALS_ID = 'jfrog-credentials'
        DOCKER_REPO = 'docker-local'
        DOCKER_IMAGE_NAME = 'myapp'
        
        // SonarQube configuration
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        
        // Java configuration
        JAVA_HOME = '/usr/lib/jvm/java-17-amazon-corretto.x86_64'
    }
    
    stages {
        stage('Check Java') {
            steps {
                sh '''
                   echo "Using JAVA_HOME=$JAVA_HOME"
                   $JAVA_HOME/bin/java -version
                   export PATH=$JAVA_HOME/bin:$PATH
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                   export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                   export PATH=$JAVA_HOME/bin:$PATH
                   mvn clean package
                '''
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh '''
                   export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                   export PATH=$JAVA_HOME/bin:$PATH
                   mvn test
                '''
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                sh '''
                   export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                   export PATH=$JAVA_HOME/bin:$PATH
                   mvn sonar:sonar \
                     -Dsonar.host.url=https://sonarcloud.io \
                     -Dsonar.login=$SONAR_TOKEN \
                     -Dsonar.projectKey=com.example:devops-project \
                     -Dsonar.organization=your-organization-key
                '''
            }
        }
        
        stage('Artifact to JFrog') {
            steps {
                rtUpload(
                    serverId: 'artifactory',
                    spec: '''{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "libs-release-local/"
                            }
                        ]
                    }'''
                )
            }
        }
        
        stage('Docker Build and Push') {
            steps {
                script {
                    // Login to JFrog Artifactory Docker registry
                    withCredentials([usernamePassword(credentialsId: "${ARTIFACTORY_CREDENTIALS_ID}", 
                                                     usernameVariable: 'ARTIFACTORY_USERNAME', 
                                                     passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        sh """
                            echo \${ARTIFACTORY_PASSWORD} | docker login ${ARTIFACTORY_URL} -u \${ARTIFACTORY_USERNAME} --password-stdin
                            
                            # Build the image with proper repository name
                            docker build -t ${ARTIFACTORY_URL}/${DOCKER_REPO}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} .
                            
                            # Push to the registry
                            docker push ${ARTIFACTORY_URL}/${DOCKER_REPO}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
                            
                            # Cleanup
                            docker rmi ${ARTIFACTORY_URL}/${DOCKER_REPO}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} || true
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh 'docker logout ${ARTIFACTORY_URL} || true'
        }
        success {
            echo 'Pipeline executed successfully'
        }
        failure {
            echo 'Pipeline execution failed'
        }
    }
}