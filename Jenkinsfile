pipeline {
  agent any
  parameters {
    string(name: 'S3_BUCKET',  defaultValue: 'oiplus')
    string(name: 'DIST_ID',    defaultValue: 'E2ZYGWK1IPRXFC')
    string(name: 'LOCAL_DIR',  defaultValue: 'web')
    string(name: 'AWS_REGION', defaultValue: 'ap-northeast-2')
  }

  stages {
    stage('Checkout') { steps { checkout scm } }   // Job의 SCM에 github-credentials 연결되어 있어야 함

    stage('Deploy') {
      steps {
        // ✅ 자격증명 ID와 파라미터 참조 수정
        withAWS(credentials: 'aws-credentials', region: "${params.AWS_REGION}") {
          sh '''
            set -e
            aws --version || { echo "[ERROR] AWS CLI not found"; exit 1; }
            test -d "${LOCAL_DIR}" || { echo "[ERROR] ${LOCAL_DIR} 폴더 없음"; exit 1; }

            echo "[DRY-RUN] S3 sync..."
            aws s3 sync "${LOCAL_DIR}/" "s3://${S3_BUCKET}/" --dryrun

            echo "[DEPLOY] S3 sync..."
            aws s3 sync "${LOCAL_DIR}/" "s3://${S3_BUCKET}/" 

            echo "[Invalidate CloudFront]"
            aws cloudfront create-invalidation --distribution-id "${DIST_ID}" --paths "/*" > cf_invalidation.json
          '''
        }
      }
    }
  }

  post {
    success { echo '✅ 배포 성공' }
    failure { echo '❌ 배포 실패 - 콘솔 확인' }
    always  { archiveArtifacts artifacts: 'cf_invalidation.json', allowEmptyArchive: true }
  }
}
