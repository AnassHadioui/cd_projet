pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    environment {
        NEXUS_URL = 'http://localhost:8081/repository/python-release/Release.zip'
    }

    stages {

        stage('Nettoyage Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Telechargement de la Release') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    bat '''
                        @echo off
                        curl.exe --fail --show-error ^
                            -u "%NEXUS_USER%:%NEXUS_PASSWORD%" ^
                            -o Release.zip ^
                            "%NEXUS_URL%"
                    '''
                }
            }
        }

        stage('Verification du telechargement') {
            steps {
                bat '''
                    if not exist Release.zip (
                        echo ERREUR : Release.zip introuvable
                        exit /b 1
                    )
                    dir Release.zip
                '''
            }
        }

        stage('Decompression de la Release') {
            steps {
                bat '''
                    powershell -Command "$ErrorActionPreference = 'Stop'; Expand-Archive -Path Release.zip -DestinationPath Release -Force"
                '''
            }
        }

        stage('Verification de la Release') {
            steps {
                bat '''
                    if not exist Release\\calculator.py exit /b 1
                    if not exist Release\\test_calculator.py exit /b 1
                    if not exist Release\\requirements.txt exit /b 1

                    dir Release
                '''
            }
        }

        stage('Deploiement DEV') {
            steps {
                bat '''
                    if not exist C:\\DEV mkdir C:\\DEV
                    xcopy Release\\* C:\\DEV\\ /E /I /Y
                '''
            }
        }

        stage('Verification DEV') {
            steps {
                bat '''
                    dir C:\\DEV
                    cd /d C:\\DEV
                    python -c "from calculator import addition; assert addition(2,3)==5; print('DEV OK')"
                '''
            }
        }

        stage('Deploiement TEST') {
            steps {
                bat '''
                    if not exist C:\\TEST mkdir C:\\TEST
                    xcopy Release\\* C:\\TEST\\ /E /I /Y
                '''
            }
        }

        stage('Tests fonctionnels') {
            steps {
                bat '''
                    cd /d C:\\TEST
                    python -m pip install -r requirements.txt
                    python -m pytest -v
                '''
            }
        }

        stage('Deploiement UAT') {
            steps {
                bat '''
                    if not exist C:\\UAT mkdir C:\\UAT
                    xcopy Release\\* C:\\UAT\\ /E /I /Y
                '''
            }
        }

        stage('Deploiement STAGING') {
            steps {
                bat '''
                    if not exist C:\\STAGING mkdir C:\\STAGING
                    xcopy Release\\* C:\\STAGING\\ /E /I /Y
                '''
            }
        }

        stage('Deploiement PRODUCTION') {
            steps {
                bat '''
                    if not exist C:\\PRODUCTION mkdir C:\\PRODUCTION
                    xcopy Release\\* C:\\PRODUCTION\\ /E /I /Y
                '''
            }
        }
    }

    post {
        success {
            echo 'Deploiement termine avec succes'
        }

        failure {
            echo 'Le pipeline a echoue'
            echo 'Verifier les logs Jenkins'
        }

        always {
            archiveArtifacts artifacts: 'Release.zip',
                             fingerprint: true,
                             allowEmptyArchive: true
        }
    }
}