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
        stage('Find Java') {
            steps {
                script {
                    // Find actual Java location
                    sh '''
                       echo "Searching for Java installation..."
                       which java
                       readlink -f $(which java) || echo "Failed to resolve java symlink"
                       ls -la /usr/lib/jvm/ || echo "JVM directory not found"
                       find /usr/lib/jvm -name "java" -type f || echo "Java executable not found in JVM directory"
                       echo "Using java version:"
                       java -version || echo "Java version check failed"
                    '''
                    
                    // Set JAVA_HOME to the actual location where Java is installed
                    sh '''
                       # Find the real path to Java executable
                       JAVA_EXEC_PATH=$(readlink -f $(which java))
                       echo "Java executable path: $JAVA_EXEC_PATH"
                       
                       # Get parent directory (bin)
                       JAVA_BIN_DIR=$(dirname "$JAVA_EXEC_PATH")
                       echo "Java bin directory: $JAVA_BIN_DIR"
                       
                       # Get parent directory of bin (this should be JAVA_HOME)
                       REAL_JAVA_HOME=$(dirname "$JAVA_BIN_DIR")
                       echo "Detected JAVA_HOME: $REAL_JAVA_HOME"
                       
                       # Export to a file that can be sourced in later stages
                       echo "export REAL_JAVA_HOME=$REAL_JAVA_HOME" > java_home.sh
                       echo "export PATH=$JAVA_BIN_DIR:$PATH" >> java_home.sh
                       chmod +x java_home.sh
                       cat java_home.sh
                    '''
                }
            }
        }
        
        stage('Debug Environment') {
            steps {
                sh '''
                   echo "==== Environment Before Sourcing ===="
                   echo "Current JAVA_HOME=$JAVA_HOME"
                   echo "Maven exists: $(which mvn || echo 'Not found')"
                   echo "====================================="
                   
                   # Source the java_home.sh file to get the REAL_JAVA_HOME
                   source ./java_home.sh
                   
                   echo "==== Environment After Sourcing ===="
                   echo "REAL_JAVA_HOME=$REAL_JAVA_HOME"
                   echo "PATH=$PATH"
                   ls -la $REAL_JAVA_HOME/bin || echo "REAL_JAVA_HOME/bin not accessible"
                   echo "====================================="
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                   # Source the java_home.sh file
                   source ./java_home.sh
                   
                   # Use the real JAVA_HOME for Maven
                   export JAVA_HOME=$REAL_JAVA_HOME
                   echo "Using JAVA_HOME=$JAVA_HOME for Maven"
                   
                   # Run Maven with the correct JAVA_HOME
                   mvn -version
                   mvn clean verify
                '''
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                sh '''
                   source ./java_home.sh
                   export JAVA_HOME=$REAL_JAVA_HOME
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