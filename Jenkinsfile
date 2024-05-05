pipeline {
    agent none

    stages {
        stage('GetCode') {
            // agent any or
                agent {label 'repo'}
            steps {
                // Obtener código del repositorio
                git branch: 'feature_fix_racecond', url: 'https://github.com/aga-unir/padevclo-cp1a.git'
                bat '''
                    hostname
                    echo %workspace%
                    whoami
                '''
                stash (name: 'repo')
            }
        }
                
        stage('Build') {
            agent {label 'build'}
            steps {
                unstash 'repo'
                bat 'tree /F'
                // Mostrar workspace
                echo WORKSPACE
                // Mostrar código descargado
                bat 'dir'
                bat '''
                    hostname
                    echo %workspace%
                    whoami
                '''
            }
        }
        
        stage ('Tests') {
            parallel {    
        
                stage('Unit') {
                    agent {label 'test'}
                    steps {
                        unstash 'repo'
                        bat 'tree /F'
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                            bat '''
                                hostname
                                echo %workspace%
                                whoami
                            '''
                             stash (name: 'report-st', includes: '*.xml') 
                        }
                    }
                }
                
                stage('Rest') {
                    agent {label 'test'}
                    steps {
                        unstash 'repo'
                        bat 'tree /F'
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set FLASK_APP=app\\api.py
                                start flask run
                                start java -jar C:\\U01-Unir\\Lab\\wiremock\\wiremock-standalone-3.5.4.jar --port 9090 --root-dir C:\\U01-Unir\\Lab\\wiremock
                                
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                            bat '''
                                hostname
                                echo %workspace%
                                whoami
                            '''
                            stash (name: 'report-nd', includes: '*.xml') 
                        }
                    }
                }
            }
        }
        
        stage('Result') {
            agent {label 'report'}
            steps {
                unstash 'report-st'
                unstash 'report-nd'
                bat 'tree /F'
                junit 'result*.xml'
                echo 'FINISH!'
                bat '''
                    hostname
                    echo %workspace%
                    whoami
                '''
            }
        }
        
        
    }
}
