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
                    def imageName = 'nodedev:v1.0'
                    bat "docker build -t ${imageName} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def port = '3001'
                    def imageName = 'nodedev:v1.0'

                    // Remove containers safely
                    bat """
                    set CONTAINERS=
                    for /F "tokens=*" %%i in ('docker ps -a -q --filter "ancestor=${imageName}"') do (
                        set CONTAINERS=%%i
                        echo Removing container %%i
                        docker rm -f %%i
                    )
                    if "%CONTAINERS%"=="" echo No containers to remove
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
