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

                // Ensure remote directory exists
                sh """
                    sshpass -p"\${WIN_PASS}" ssh -o StrictHostKeyChecking=no \${WIN_USER}@${windowsHost} powershell -Command "if (!(Test-Path '${remoteDir}')) { New-Item -ItemType Directory -Path '${remoteDir}' }"
                """

                // Copy artifact to Windows
                sh """
                    sshpass -p"\${WIN_PASS}" scp -o StrictHostKeyChecking=no ${artifact} \${WIN_USER}@${windowsHost}:'${remoteDir}/'
                """

                // Extract just the file name (without path)
                def artifactName = artifact.tokenize('/').last()

                // Stop old Java process (if running) and start the new JAR
                sh """
                    sshpass -p"\${WIN_PASS}" ssh -o StrictHostKeyChecking=no \${WIN_USER}@${windowsHost} powershell -Command "
                        Get-Process java -ErrorAction SilentlyContinue | Stop-Process -Force;
                        Start-Process java -ArgumentList '-jar ${remoteDir}/${artifactName}' -WindowStyle Hidden
                    "
                """
            }
        }
    }
}

stage('Verify Deployment') {
    steps {
        script {
            def windowsHost = "10.81.234.130"

            withCredentials([usernamePassword(credentialsId: 'windows-creds', usernameVariable: 'WIN_USER', passwordVariable: 'WIN_PASS')]) {
                sh """
                    sshpass -p"\${WIN_PASS}" ssh -o StrictHostKeyChecking=no \${WIN_USER}@${windowsHost} powershell -Command "
                        if (Get-Process java -ErrorAction SilentlyContinue) {
                            Write-Output '✅ Application is running.'
                        } else {
                            Write-Output '❌ Application failed to start.'
                            exit 1
                        }
                    "
                """
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
