pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                // Obtener código del repositorio
                git 'https://github.com/aga-unir/padevclo-cp1a.git'
                
            }
        }
        
        stage ('Static'){
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        flake8 --format=pylint --exit-zero app >flake8.out
                    '''
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold:8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                }
            }
        }
                       
        stage ('Tests') {
            parallel {    
        
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                                coverage xml -o cov-unit.xml
                            '''
                            junit 'result-unit.xml'
                            stash (name: 'cov_unit', includes: 'cov-unit.xml') 
                        }
                    }
                }
                
                stage('Rest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set FLASK_APP=app\\api.py
                                start flask run & sleep 5
                                start java -jar C:\\U01-Unir\\Lab\\wiremock\\wiremock-standalone-3.5.4.jar --port 9090 --root-dir test\\wiremock --verbose & sleep 30
                                
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                            junit 'result-rest.xml'
                        }
                    }
                }
            }
        }
        
        stage ('Security'){
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}“
                    '''
                     recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold:2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                }
            }
        }
        
        stage ('Cobertura'){
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    unstash 'cov_unit'
                    cobertura coberturaReportFile:'cov-unit.xml', conditionalCoverageTargets:'100,90,80', lineCoverageTargets: '100,95,85', onlyStable: false, failUnstable: false
                }
            }
        }
        
        stage ('Performance'){
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set FLASK_APP=app\\api.py
                        curl localhost:5000/stopServer
                        sleep 20 & start flask run & sleep 10
                    '''
                    bat 'C:\\U01-Unir\\Lab\\apache-jmeter-5.6.3\\bin\\jmeter.bat -n -t test\\jmeter\\flask.jmx -f -l flask.jtl'
                    perfReport sourceDataFiles: 'flask.jtl'
                }
            }
        }
    }
}
