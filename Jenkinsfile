pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN') // This must match the ID in Jenkins credentials
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building project...'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'echo "Running Sonar analysis..."'
                    sh 'mvn verify sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }
    }
}
