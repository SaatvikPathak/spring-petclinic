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
    
   stage('Deploy to Windows') {
    steps {
        script {
            def windowsHost = "10.81.234.130"
            def remoteDir = "C:/deployments/myapp"

            withCredentials([usernamePassword(credentialsId: 'windows-creds', usernameVariable: 'WIN_USER', passwordVariable: 'WIN_PASS')]) {
                // Find the first JAR in target/ directory
                def artifact = sh(script: "ls target/*.jar | head -n 1", returnStdout: true).trim()
                echo "Detected artifact: ${artifact}"
                def artifactName = artifact.tokenize('/').last()

                // Inject env vars so we don’t interpolate directly
                withEnv([
                    "WIN_HOST=${windowsHost}",
                    "REMOTE_DIR=${remoteDir}",
                    "ARTIFACT=${artifact}",
                    "ARTIFACT_NAME=${artifactName}"
                ]) {

                    // Ensure remote directory exists
                    sh '''
                        sshpass -p"$WIN_PASS" ssh -o StrictHostKeyChecking=no $WIN_USER@$WIN_HOST powershell -Command "if (!(Test-Path `$Env:REMOTE_DIR)) { New-Item -ItemType Directory -Path `$Env:REMOTE_DIR }"
                    '''

                    // Copy artifact to Windows
                    sh '''
                        sshpass -p"$WIN_PASS" scp -o StrictHostKeyChecking=no "$ARTIFACT" $WIN_USER@$WIN_HOST:"$REMOTE_DIR"/
                    '''

                    // Run the app remotely
                    sh '''
                        sshpass -p"$WIN_PASS" ssh -o StrictHostKeyChecking=no $WIN_USER@$WIN_HOST powershell -Command "Start-Process java -ArgumentList '-jar `$Env:REMOTE_DIR/`$Env:ARTIFACT_NAME' -WindowStyle Hidden"
                    '''
                }
            }
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
