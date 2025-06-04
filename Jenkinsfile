pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-northeast-2'
        AWS_ACCESS_KEY_ID = credentials('ecr-login')
        AWS_SECRET_ACCESS_KEY = credentials('ecr-login')
        ECR_REGISTRY = '341162387145.dkr.ecr.ap-northeast-2.amazonaws.com'
        APP_REPO_NAME = 'nsa'
        SEMGREP_API_TOKEN = 'de92eecd94c6ceb9b4d6abb49b684e28a3717011b567b4bc58f3dd572770760b'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'develop',
                    url: 'https://github.com/nsa0320/bWAPP.git',
                    credentialsId: '1'
            }
        }

        stage('Semgrep Cloud API Scan') {
            steps {
                script {
                    def archiveName = "semgrep-src.tar.gz"
                    def tempDir = "/tmp/semgrep-src"

                    sh """
                        echo "[📁] 코드 임시 복사..."
                        rm -rf ${tempDir}
                        mkdir -p ${tempDir}
                        cp -r . ${tempDir} || true

                        echo "[📦] 압축 생성 중..."
                        tar --exclude='.git' --exclude='target' --exclude='uploads' --exclude='logs' --exclude="${archiveName}" -czf ${archiveName} -C ${tempDir} .

                        echo "[🔐] Semgrep Cloud API 요청..."
                        curl -s -X POST https://semgrep.dev/api/v1/scans \\
                          -H "Authorization: Bearer $SEMGREP_API_TOKEN" \\
                          -F "scan=@${archiveName}" > semgrep-api-response.json || echo '{ "error": "upload_failed" }' > semgrep-api-response.json

                        echo "[📄] API 응답:"
                        cat semgrep-api-response.json
                    """

                    // scan_id 추출해서 보기 좋게 출력
                    def result = readJSON file: 'semgrep-api-response.json'
                    if (result.scan_id) {
                        echo "✅ Semgrep Scan ID: ${result.scan_id}"
                        echo "🔎 확인 링크: https://semgrep.dev/scans/${result.scan_id}"
                    } else {
                        error("❌ Semgrep 분석 요청 실패! 응답 확인 필요.")
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build --force-rm -t $ECR_REGISTRY/$APP_REPO_NAME:latest .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push $ECR_REGISTRY/$APP_REPO_NAME:latest'
            }
        }
    }

    post {
        always {
            echo '🧹 Docker 이미지 정리 중...'
            sh 'docker image prune -af || true'
        }
        success {
            echo '✅ 파이프라인 성공!'
        }
        failure {
            echo '❌ 파이프라인 실패. 로그 확인 필요!'
        }
    }
}
