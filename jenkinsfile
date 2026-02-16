pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rohith13s/petclinic"   // change if needed
    }

    stages {

        stage('Clone') {
            steps {
                echo "Cloning repository..."
            }
        }

        stage('Build Application') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                usernameVariable: 'USERNAME',
                passwordVariable: 'PASSWORD')]) {

                    sh """
                    docker login -u $USERNAME -p $PASSWORD
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy Green Environment') {
            steps {
                sh """
                sed 's/IMAGE_TAG/${BUILD_NUMBER}/g' k8s/green-deployment.yaml | kubectl apply -f -
                kubectl rollout status deployment/petclinic-green
                """
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh """
                kubectl patch service petclinic-service \
                -p '{"spec":{"selector":{"app":"petclinic","version":"green"}}}'
                """
            }
        }
    }

    post {
        success {
            echo "Deployment Successful 🚀"
        }
        failure {
            echo "Build Failed ❌"
        }
    }
}

