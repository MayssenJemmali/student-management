pipeline {
    agent any

    tools {
        maven 'M2_HOME'  // Make sure this matches the Maven installation name in Jenkins 
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

        stage('SonarQube') {
            steps {
                echo '🔍 Running SonarQube analysis...'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=student \
                          -Dsonar.login=$SONAR_TOKEN \
                          -Dsonar.java.binaries=target/classes
                    '''
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
        stage('Deploy to Kubernetes') {
      steps {
        sh '''
          kubectl version --client
          kubectl get nodes

          # Apply manifests
          kubectl apply -f k8s/namespace.yaml
          kubectl apply -f k8s/mysql-deployment.yaml
          kubectl apply -f k8s/mysql-service.yaml
          kubectl apply -f k8s/backend-deployment.yaml
          kubectl apply -f k8s/backend-service.yaml

          # Wait for rollouts
          kubectl rollout status deployment/mysql -n ${K8S_NAMESPACE} --timeout=180s
          kubectl rollout status deployment/student-backend -n ${K8S_NAMESPACE} --timeout=240s

          # Show status
          kubectl get pods -n ${K8S_NAMESPACE} -o wide
          kubectl get svc -n ${K8S_NAMESPACE}
        '''
      }
    }

    post {
        success { echo "✅ Pipeline succeeded" }
        unstable { echo "⚠️ Some tests failed (UNSTABLE)" }
        failure { echo "❌ Pipeline failed" }
    }
}
