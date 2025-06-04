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
                    def archiveName = "semgrep-src.tar.gz"

                    sh """
                        echo "[📦] 코드 압축 중..."
                        # archiveName 자신을 압축 대상에서 제외합니다.
                        tar --exclude='.git' --exclude='target' --exclude="${archiveName}" -czf ${archiveName} .

                        echo "[🔐] Semgrep Cloud API 호출..."
                        curl -X POST https://semgrep.dev/api/v1/scans \
                          -H "Authorization: Bearer $SEMGREP_APP_TOKEN" \
                          -F "scan=@${archiveName}" > semgrep-api-response.json

                        echo "[📄] 응답 저장 완료: semgrep-api-response.json"
                    """
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
