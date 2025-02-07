pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git branch: 'dev', url: 'https://github.com/cedriclecalvez/todo-list-aws-devops-unir.git'
            }
        }

        stage('Static Tests') {
            parallel {
                stage('Flake8') {
                    steps {
                        sh '''
                            python3 -m flake8 --exit-zero --format=pylint src >flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                    }
                }

                stage('Security') {
                    steps {
                        sh '''
                            python3 -m bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        '''
                        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                    sh '''
                    export AWS_REGION=us-east-1
                    export AWS_DEFAULT_REGION=us-east-1
                    rm -rf samconfig.toml
                    rm -rf temlplate.yaml
                    sam build
                    sam validate --region us-east-1
                     sam deploy --stack-name todo-list-aws-staging \
                       --resolve-s3 \
                       --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
                       --region us-east-1 \
                       --parameter-overrides Stage="staging" AllowUnauthenticated="true" \
                       --no-confirm-changeset
                '''
            }
        }

        // stage('Unit') {
        //     steps {
        //         sh '''
        //             export PYTHONPATH=%WORKSPACE%
        //             python3 -m pytest --junitxml=result-unit.xml test/unit
        //         '''
        //         junit 'result-unit.xml'
        //     }
        // }

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
