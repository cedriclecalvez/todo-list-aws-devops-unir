pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git branch: 'dev', url: 'https://github.com/cedriclecalvez/todo-list-aws-devops-unir.git'
                sh '''
                wget https://raw.githubusercontent.com/cedriclecalvez/todo-list-aws-config_devops-unir/staging/samconfig.toml -O samconfig.toml
                '''
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
            // steps {
            //         sh '''
            //         export AWS_REGION=us-east-1
            //         export AWS_DEFAULT_REGION=us-east-1
            //         rm -rf .aws-sam samconfig.toml
            //         sam build
            //         sam validate --region us-east-1
            //         sam deploy --stack-name todo-list-aws-staging \
            //            --resolve-s3 \
            //            --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
            //            --region us-east-1 \
            //            --parameter-overrides Stage="staging" \
            //            --no-confirm-changeset \
            //            --force-upload \
            //            --no-fail-on-empty-changeset
            //     '''
            // }
            steps {
                sh '''
                export AWS_REGION=us-east-1
                export AWS_DEFAULT_REGION=us-east-1
                rm -rf .aws-sam
                sam build
                sam validate --region us-east-1
                sam deploy --config-env staging --no-confirm-changeset --force-upload --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Tests') {
            steps {
                script {
                    def BASE_URL = sh(script: "aws cloudformation describe-stacks --stack-name todo-list-aws-production --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text",
                    returnStdout: true).trim()
                    echo "API Base URL: ${BASE_URL}"
                    echo 'Initiating Integration Tests'
                    sh """
                        export BASE_URL="${BASE_URL}"
                        python3 -m pytest --junitxml=result-integration.xml -m "readonly" test/integration/todoApiTest.py
                    """
                }
                junit 'result-integration.xml'
            }
        }
        stage('Promote') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_PAT')]) {
                        sh '''
                            git config user.email "jenkins@ci.local CP1.4"
                            git config user.name "Jenkins CI Cedric CP1.4"
                            git remote set-url origin https://$GITHUB_PAT@github.com/cedriclecalvez/todo-list-aws-devops-unir.git
                            git checkout master
                            git pull origin master
                            git merge --no-ff dev -m "Promoting version from dev to master" || {
                                echo "Merge conflict detected. Attempting to resolve automatically."
                                git merge --abort
                                git merge --strategy-option theirs dev || {
                                    echo "Automatic resolution failed. Aborting merge."
                                    git merge --abort
                                    exit 1
                                }
                            }
                            git push origin master
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/*.xml', fingerprint: true
            echo 'Artifacts archived.'
        }
        success {
            cleanWs()
            echo 'Cleaning done'
            echo 'Pipeline completed successfully!'
        }
        failure {
            cleanWs()
            echo 'Cleaning done'
            echo 'Pipeline failed.'
        }
    }
}
