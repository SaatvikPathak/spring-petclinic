pipeline {
    agent any

    tools {
        maven 'Maven 3.9'    // Configure Maven in Jenkins
        jdk 'Java 21'      // Configure JDK in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SaatvikPathak/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Code Coverage') {
      steps {
        jacoco execPattern: '**/target/jacoco.exec'
      }
    }
    
     stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarQube' // Name from Global Tool Config
            }
            steps {
                withSonarQubeEnv('MySonarQube') {  // Name from SonarQube server config
                    bat '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=my-app \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

    }

    post {
        success {
            echo '✅ Build and tests passed!'
        }
        failure {
            echo '❌ Build failed! Check logs.'
        }
    }
} 
