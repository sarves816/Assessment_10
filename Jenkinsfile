
pipeline {
    agent any

    parameters {
        choice(
            name: 'BUILD_MODE',
            choices: ['FULL', 'TEST_ONLY'],
            description: 'Select build mode'
        )
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'v1',
            description: 'Docker image tag'
        )
    }

    environment {
        IMAGE = 'industrial-maintenance'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
                echo 'GitHub checkout completed.'
            }
        }

        stage('Parallel Validation') {
            parallel {
                stage('HTML Validation') {
                    steps {
                        bat '''
                        if not exist index.html (
                            echo ERROR: index.html not found
                            exit /b 1
                        )
                        findstr /C:"<html" index.html
                        if errorlevel 1 exit /b 1
                        findstr /C:"</html>" index.html
                        if errorlevel 1 exit /b 1
                        echo HTML validation successful.
                        '''
                    }
                }

                stage('Dockerfile Validation') {
                    steps {
                        bat '''
                        if not exist Dockerfile (
                            echo ERROR: Dockerfile not found
                            exit /b 1
                        )
                        findstr /C:"FROM nginx" Dockerfile
                        if errorlevel 1 exit /b 1
                        echo Dockerfile validation successful.
                        '''
                    }
                }
            }
        }

        stage('Build and Test') {
            when {
                expression {
                    params.BUILD_MODE == 'FULL'
                }
            }
            steps {
                bat '''
                docker version
                docker build -t %IMAGE%:%IMAGE_TAG% .
                docker rm -f maintenance-test 2>NUL
                docker run -d --name maintenance-test -p 18080:80 %IMAGE%:%IMAGE_TAG%
                if errorlevel 1 exit /b 1

                timeout /t 5 /nobreak

                curl.exe --fail http://localhost:18080/
                if errorlevel 1 (
                    docker logs maintenance-test
                    docker rm -f maintenance-test
                    exit /b 1
                )

                docker rm -f maintenance-test
                echo Docker build and test successful.
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts(
                    artifacts: 'index.html,Dockerfile,Jenkinsfile,deployment.yaml',
                    allowEmptyArchive: false
                )
                echo 'Project files archived successfully.'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Pipeline completed.'
        }
        failure {
            echo 'FAILURE: Check the Console Output for the failed stage.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
