pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.9'
        jdk 'OpenJDK 11' // Using the available JDK in Jenkins
    }
    
    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        // Explicitly define JAVA_HOME here
        JAVA_HOME = tool 'OpenJDK 11' // Using the available JDK in Jenkins
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
                withEnv(["JAVA_HOME=${tool 'OpenJDK 11'}", "PATH+MAVEN=${tool 'OpenJDK 11'}/bin:${tool 'Maven 3.9.9'}/bin"]) {
                    sh 'mvn clean verify'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withEnv(["JAVA_HOME=${tool 'OpenJDK 11'}", "PATH+MAVEN=${tool 'OpenJDK 11'}/bin:${tool 'Maven 3.9.9'}/bin"]) {
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