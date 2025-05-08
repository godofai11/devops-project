pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.9'
        // Remove JDK tool reference to use the system Java
    }
    
    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        JAVA_HOME = '/usr/lib/jvm/java-17-amazon-corretto.x86_64'  // Set JAVA_HOME directly
    }
    
    stages {
        stage('Check Java') {
            steps {
                sh '''
                   echo "Using JAVA_HOME=$JAVA_HOME"
                   $JAVA_HOME/bin/java -version
                   export PATH=$JAVA_HOME/bin:$PATH
                   echo "PATH=$PATH"
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                   # Ensure Java 17 is in the PATH
                   export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                   export PATH=$JAVA_HOME/bin:$PATH
                   
                   # Run Maven with the correct JAVA_HOME
                   mvn -version
                   mvn clean verify
                '''
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                sh '''
                   export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                   export PATH=$JAVA_HOME/bin:$PATH
                   mvn sonar:sonar -Dsonar.host.url=http://your-sonarqube-url -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Docker Build and Push') {
            steps {
                sh 'docker build -t your-image-name:$BUILD_NUMBER .'
                sh 'docker tag your-image-name:$BUILD_NUMBER your-registry/your-image-name:$BUILD_NUMBER'
                sh 'docker push your-registry/your-image-name:$BUILD_NUMBER'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
        }
        success {
            echo 'Pipeline executed successfully'
        }
        failure {
            echo 'Pipeline execution failed'
        }
    }
}