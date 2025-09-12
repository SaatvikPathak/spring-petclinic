pipeline {
    agent any

    tools {
        maven 'Maven'    // Configure Maven in Jenkins
        jdk 'Java17'      // Configure JDK in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SaatvikPathak/spring-petclinic.git'
            }
        }
        
        stage('Format Code') {
    steps {
        sh 'mvn spring-javaformat:apply'
    }
}
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Code Coverage') {
      steps {
        jacoco execPattern: '**/target/jacoco.exec'
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
