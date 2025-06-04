pipeline {
    agent any

    environment {
        SEMGREP_APP_TOKEN = credentials('SEMGREP_APP_TOKEN')
        ARCHIVE_NAME = "semgrep-src.tar.gz"
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

        stage('Semgrep Cloud API Scan') {
            steps {
                script {
                    def START = System.currentTimeMillis()

                    sh '''
                        echo "[📦] 코드 압축 중..."
                        tar --exclude='.git' --exclude='target' -czf ${ARCHIVE_NAME} .

                        echo "[🔐] Semgrep Cloud API 호출..."
                        curl -X POST https://semgrep.dev/api/v1/scans \
                          -H "Authorization: Bearer $SEMGREP_APP_TOKEN" \
                          -F "scan=@${ARCHIVE_NAME}" > semgrep-api-response.json

                        echo "[📄] Semgrep 응답 저장 완료: semgrep-api-response.json"
                    '''

                    def END = System.currentTimeMillis()
                    def durationSeconds = (END - START) / 1000.0
                    echo "⏱️ Semgrep 분석 총 소요 시간: ${durationSeconds}초"
                }
            }
        }

        stage('Visualize Semgrep Result') {
            steps {
                sh '''
                    echo "[📄] HTML 리포트 생성 중..."
                    python3 create_semgrep_report.py semgrep-api-response.json
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
