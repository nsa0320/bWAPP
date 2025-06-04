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

            sh """
                echo "[📦] 코드 압축 중..."
                # archiveName 자신을 압축 대상에서 제외합니다.
                tar --exclude='.git' --exclude='target' --exclude="${archiveName}" -czf ${archiveName} .

                echo "[🔐] Semgrep Cloud API 호출..."
                curl -X POST https://semgrep.dev/api/v1/scans \
                  -H "Authorization: Bearer $SEMGREP_API_TOKEN" \
                  -F "scan=@${archiveName}" > semgrep-api-response.json

                echo "[📄] 응답 저장 완료: semgrep-api-response.json"
            """
        }
    }
}

        stage('Build Docker Image') {
            steps {
                sh 'docker build --force-rm -t $ECR_REGISTRY/$APP_REPO_NAME:latest .'
            }
        }

    post {
        always {
            echo '🧹 Docker 이미지 정리 중...'
            sh 'docker image prune -af'
        }
        success {
            echo '✅ 파이프라인 성공!'
        }
        failure {
            echo '❌ 파이프라인 실패. 로그 확인 필요!'
        }
    }
}
