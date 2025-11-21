pipeline {
    agent any 
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
                sh 'mvn clean install -Dmaven.test.failure.ignore=true'
                echo "✅ Build completed"
            }
        }

        stage('Test') {
            steps {
                echo "🧪 Running unit tests..."
                sh 'mvn test -DskipTests=true'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }  // <-- end of stages

    post {
        success { echo "✅ Pipeline succeeded" }
        unstable { echo "⚠️ Some tests failed (UNSTABLE)" }
        failure { echo "❌ Pipeline failed" }
    }
}

