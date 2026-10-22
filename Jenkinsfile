pipeline {
    agent any

    parameters {
        choice(name: 'BUILD_MODE',
               choices: ['FULL', 'TEST_ONLY'],
               description: 'Select pipeline mode')
        string(name: 'IMAGE_TAG',
               defaultValue: 'v1',
               description: 'Docker image tag')
    }

    environment {
        IMAGE = 'sarves816/industrial-maintenance'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sarves816/Assessment_10.git'
            }
        }

        stage('Parallel Validation') {
            parallel {
                stage('HTML Validation') {
                    steps {
                        sh '''
                        test -s index.html
                        grep -q "<html" index.html
                        grep -q "</html>" index.html
                        '''
                    }
                }

                stage('Dockerfile Validation') {
                    steps {
                        sh '''
                        test -s Dockerfile
                        grep -q "FROM nginx" Dockerfile
                        '''
                    }
                }
            }
        }

        stage('Build and Test') {
            when {
                expression { params.BUILD_MODE == 'FULL' }
            }
            steps {
                sh '''
                docker build -t $IMAGE:$IMAGE_TAG .
                docker run -d --name maintenance-test -p 18080:80 $IMAGE:$IMAGE_TAG
                sleep 3
                curl --fail http://localhost:18080/
                docker rm -f maintenance-test
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'index.html,Dockerfile,Jenkinsfile,deployment.yaml',
                                 allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Build and validation successful.'
        }
        failure {
            echo 'Build or validation failed. Check the console output.'
        }
    }
}


