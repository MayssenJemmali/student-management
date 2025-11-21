pipeline {
    agent any 
    stages{
        stage('Checkout'){
            steps{
                echo "🔁 Checking out repository..."
                git branch: 'main', url: 'https://github.com/MayssenJemmali/student-management.git'
                // quick verification commands (will appear in the build console)
                sh 'echo "Current branch:" && git rev-parse --abbrev-ref HEAD'
                sh 'echo "Last commit:" && git log -1 --oneline'
                sh 'echo "Workspace files:" && ls -la'
            }
        }

        stage('Build'){
            steps{
                echo "🔨 Building project with Maven..."
                withMaven(maven: 'M2_HOME') {
                  sh 'mvn clean install -DskipTests'
                }
                echo "✅ Build completed"
            }
        }
    }
}
