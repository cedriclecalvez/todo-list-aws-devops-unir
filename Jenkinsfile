pipeline {
    agent any

     stages {
        stage('Get Code') {
            steps {
                git branch: 'dev', url: 'https://github.com/cedriclecalvez/todo-list-aws-devops-unir.git'
            }
        }

        stage('Flake8') {
            steps {
                sh '''
                    python -m flake8 --exit-zero --format=pylint app >flake8.out
                '''   
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            }
        }

        stage('Security') {
            steps {
                sh '''
                    python -m bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
            }
        }        
        
        stage('Unit') {
            steps {
                sh '''
                    export PYTHONPATH=%WORKSPACE%
                    python -m pytest --junitxml=result-unit.xml test/unit
                '''
                junit 'result-unit.xml'
            }
        }

        // stage('Rest') {
        //     steps {
        //         sh '''
        //             export FLASK_APP=app/api.py
        //             start flask run -p 5001
        //             start java -jar C:/Unir/Ejercicios/wiremock-jre8-standalone-2.28.0.jar --port 9090 --root-dir test/wiremock
        //             export PYTHONPATH=.
        //             ping 127.0.0.1 -n 15
        //             pytest --junitxml=result-rest.xml test/rest
        //         '''
        //         junit 'result-rest.xml'
        //     }    
        // }

        // stage('Cobertura') {
        //     steps {
        //         sh '''
        //             coverage xml
        //         '''
        //         catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
        //             cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,85', onlyStable: false, failUnstable: false
        //         }
        //     }
        // }

        // stage('Performance') {
        //     steps {
        //         sh '''
        //             export FLASK_APP=app/api.py
        //             export FLASK_ENV=development
        //             start flask run

        //             ping 127.0.0.1 -n 15
                    
        //             C:/UNIR/Ejercicios/apache-jmeter-5.5/bin/jmeter -n -t test/jmeter/flask.jmx -f -l flask.jtl
        //         '''
        //         perfReport sourceDataFiles: 'flask.jtl'
        //     }
        // }  
    }
}