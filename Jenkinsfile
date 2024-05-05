pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                // Obtener código del repositorio
                git branch: 'develop', url: 'https://github.com/aga-unir/padevclo-cp1a.git'                
            }
        }
                
        stage('Build') {
            steps {
                // Mostrar workspace
                echo WORKSPACE
                // Mostrar código descargado
                bat 'dir'
            }
        }
        
        stage ('Tests') {
            parallel {    
        
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
                
                stage('Rest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set FLASK_APP=app\\api.py
                                start flask run
                                start java -jar C:\\U01-Unir\\Lab\\wiremock\\wiremock-standalone-3.5.4.jar --port 9090 --root-dir C:\\U01-Unir\\Lab\\wiremock 
                                
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Result') {
            steps {
                junit 'result*.xml'
                echo 'FINISH!'
            }
        }
        
        
    }
}
