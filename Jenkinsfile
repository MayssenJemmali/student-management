pipeline {
    agent any

    tools {
        maven 'M2_HOME'  // Make sure this matches the Maven installation name in Jenkins ss
    }

    environment {
        DOCKERHUB_USER = 'mayssenj'
        IMAGE_NAME     = 'student-management'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔁 Checking out repository..."
                git branch: 'main', url: 'https://github.com/MayssenJemmali/student-management.git'
                sh 'echo "Current branch:" && git rev-parse --abbrev-ref HEAD'
                sh 'echo "Last commit:" && git log -1 --oneline'
                sh 'echo "Workspace files:" && ls -la'
            }
        }

        stage('Build & Install') {
            steps {
                echo "🔨 Building project with Maven..."
                sh 'mvn clean install'
                echo "✅ Build completed"
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Running unit tests..."
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    echo "📊 Test results published"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "🚀 Pushing Docker image to DockerHub..."
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',  // Replace with your Jenkins DockerHub credentials ID
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success { echo "✅ Pipeline succeeded" }
        unstable { echo "⚠️ Some tests failed (UNSTABLE)" }
        failure { echo "❌ Pipeline failed" }
    }
}
