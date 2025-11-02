pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                bat 'npm install'
            }
        }

        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Test') {
            steps {
                bat 'npm test --watchAll=false'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Use different images for main vs other branches
                    def imageName = (env.BRANCH_NAME == 'main') ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    bat "docker build -t ${imageName} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def port = (env.BRANCH_NAME == 'main') ? '3000' : '3001'
                    def imageName = (env.BRANCH_NAME == 'main') ? 'nodemain:v1.0' : 'nodedev:v1.0'

                    // Remove containers of this image safely
                    bat """
                    for /F "tokens=*" %%i in ('docker ps -a -q --filter "ancestor=${imageName}"') do (
                        echo Removing container %%i
                        docker rm -f %%i
                    )
                    """

                    // Run container
                    bat "docker run -d -p ${port}:${port} ${imageName}"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo "Deployment successful for branch ${env.BRANCH_NAME}!"
        }
        failure {
            echo "Pipeline failed for branch ${env.BRANCH_NAME}."
        }
    }
}
