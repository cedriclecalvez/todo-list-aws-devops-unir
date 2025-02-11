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
                    rm -rf .aws-sam samconfig.toml
                    sam build
                    sam validate --region us-east-1
                    sam deploy --stack-name todo-list-aws-staging \
                       --resolve-s3 \
                       --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
                       --region us-east-1 \
                       --parameter-overrides Stage="staging" \
                       --no-confirm-changeset \
                       --force-upload \
                       --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Tests') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    script {
                        def BASE_URL = sh(script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text",
                        returnStdout: true)
                        // def BASE_URL = sh(script:  "aws cloudformation describe - stacks - stack - name todo - list - aws - staging \
                        //     --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                        //     -- region us - east - 1 - output text", returnStdout: true)
                        echo "API Base URL: ${BASE_URL}"
                        echo 'Initiating Integration Tests'
                        sh "bash test/integration/integration.sh $BASE_URL"
                        sh "BASE_URL=${BASE_URL} pytest --junitxml=result-integration.xml test/integration"
                    }
                //     sh '''
                //     export PYTHONPATH=${WORKSPACE}
                //     python3 -m pytest --junitxml=result-unit.xml test/integration
                // '''
                }

                junit 'result-unit.xml'
            }
        }
    }
}
