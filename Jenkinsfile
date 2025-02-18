pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git branch: 'master', url: 'https://github.com/cedriclecalvez/todo-list-aws-devops-unir.git'
                sh '''
                wget https://raw.githubusercontent.com/cedriclecalvez/todo-list-aws-config_devops-unir/production/samconfig.toml -O samconfig.toml
                '''
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
            //         sam deploy --stack-name todo-list-aws-production \
            //            --resolve-s3 \
            //            --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
            //            --region us-east-1 \
            //            --parameter-overrides Stage="production" \
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
                sam deploy --config-env production --no-confirm-changeset --force-upload --no-fail-on-empty-changeset
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
