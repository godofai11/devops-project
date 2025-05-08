pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.9'
        jdk 'OpenJDK 11'
    }
    
    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }
    
    stages {
        stage('Set Up Environment') {
            steps {
                script {
                    // Get absolute paths from tool steps
                    def javaHome = tool 'OpenJDK 11'
                    def mvnHome = tool 'Maven 3.9.9'
                    
                    // Print debug info
                    sh "echo Java home: ${javaHome}"
                    sh "echo Maven home: ${mvnHome}"
                    
                    // Set environment variables for all steps using environment directive
                    env.JAVA_HOME = javaHome
                    env.M2_HOME = mvnHome
                    env.PATH = "${javaHome}/bin:${mvnHome}/bin:${env.PATH}"
                    
                    sh "echo Updated JAVA_HOME: ${env.JAVA_HOME}"
                    sh "echo Updated PATH: ${env.PATH}"
                }
            }
        }
        
        stage('Debug Environment') {
            steps {
                sh '''
                   echo "==== Environment Debugging ===="
                   echo "JAVA_HOME=$JAVA_HOME"
                   echo "M2_HOME=$M2_HOME"
                   echo "PATH=$PATH"
                   which java || echo "java not found in PATH"
                   java -version || echo "java -version failed"
                   which mvn || echo "mvn not found in PATH"
                   mvn -version || echo "mvn -version failed"
                   ls -la $JAVA_HOME/bin || echo "JAVA_HOME/bin not accessible"
                   ls -la $M2_HOME/bin || echo "M2_HOME/bin not accessible"
                   echo "===== End Debugging ====="
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                   export JAVA_HOME=$JAVA_HOME
                   export PATH=$JAVA_HOME/bin:$PATH
                   echo "Build step JAVA_HOME=$JAVA_HOME"
                   echo "Build step PATH=$PATH"
                   mvn clean verify
                '''
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                sh '''
                   export JAVA_HOME=$JAVA_HOME
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