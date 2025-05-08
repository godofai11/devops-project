pipeline {
    agent any
    
    tools {
        maven 'Maven_3.9.9'
        jdk 'JDK17' // Make sure this matches your Jenkins tool configuration
    }
    
    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        // Explicitly define JAVA_HOME here
        JAVA_HOME = tool 'JDK17' // Must match the tool name in Jenkins
    }
    
    stages {
        stage('Set Up Environment') {
            steps {
                script {
                    sh "echo JAVA_HOME=$JAVA_HOME"
                    sh "echo PATH=$PATH"
                }
            }
        }
        
        stage('Debug Environment') {
            steps {
                sh 'echo JAVA_HOME=$JAVA_HOME'
                sh 'echo PATH=$PATH'
                sh 'which java'
                sh 'java -version'
                sh 'mvn -version || echo "mvn -version failed"'
            }
        }
        
        stage('Build') {
            steps {
                withEnv(["JAVA_HOME=${tool 'JDK17'}", "PATH+MAVEN=${tool 'JDK17'}/bin:${tool 'Maven_3.9.9'}/bin"]) {
                    sh 'mvn clean verify'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withEnv(["JAVA_HOME=${tool 'JDK17'}", "PATH+MAVEN=${tool 'JDK17'}/bin:${tool 'Maven_3.9.9'}/bin"]) {
                    sh 'mvn sonar:sonar -Dsonar.host.url=http://your-sonarqube-url -Dsonar.login=$SONAR_TOKEN'
                }
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
}