pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-northeast-2'
        AWS_ACCESS_KEY_ID = credentials('ecr-login')
        AWS_SECRET_ACCESS_KEY = credentials('ecr-login')
        S3_BUCKET = 'webgoat-nsa'
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git branch: 'develop',
                    url: 'https://github.com/nsa0320/bWAPP.git',
                    credentialsId: '1'
            }
        }

        stage('Semgrep Analysis via Lambda') {
            steps {
                script {
                    def START = System.currentTimeMillis()

                    sh '''
                        echo "[📦] 소스코드 압축 중..."
                        zip -r source.zip . -x "*.git*" "*.idea*" "target/*"

                        echo "[☁️] S3에 업로드 중..."
                        aws s3 cp source.zip s3://$S3_BUCKET/source.zip

                        echo "[🚀] Lambda로 Semgrep 실행 요청 중..."
                        aws lambda invoke \
                          --function-name trigger-semgrep-analysis-ssm \
                          --payload '{"s3_key":"source.zip"}' \
                          --region $AWS_REGION \
                          --cli-binary-format raw-in-base64-out \
                          lambda_output.json

                        echo "[📄] Lambda 응답 내용:"
                        cat lambda_output.json
                    '''

                    def END = System.currentTimeMillis()
                    def durationSeconds = (END - START) / 1000.0
                    echo "⏱️ Semgrep 분석 총 소요 시간: ${durationSeconds}초"
                }
            }
        }

        stage('Download and Visualize Semgrep Result') {
            steps {
                sh '''
                    echo "[📥] S3에서 Semgrep 결과 다운로드..."
                    aws s3 cp s3://$S3_BUCKET/semgrep-result.json semgrep-result.json

                    echo "[📄] HTML 리포트 생성 중..."
                    python3 create_semgrep_report.py
                '''
            }
        }

        stage('Publish Semgrep Report') {
            steps {
                publishHTML([
                    reportDir: '.', 
                    reportFiles: 'semgrep-report.html', 
                    reportName: 'Semgrep 분석 리포트',
                    keepAll: true,
                    alwaysLinkToLastBuild: true,
                    allowMissing: false
                ])
            }
        }
    }

    post {
        success {
            echo '✅ Semgrep 리포트 생성 완료!'
        }
        failure {
            echo '❌ 실패! 로그 확인 필요.'
        }
    }
}
