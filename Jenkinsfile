pipeline {
  agent any
  parameters {
    string(name: 'S3_BUCKET', defaultValue: 'hsj-project2')
    string(name: 'DIST_ID',   defaultValue: 'E6S76RL7FIVT2')
    string(name: 'LOCAL_DIR', defaultValue: 'web')
    string(name: 'AWS_REGION',defaultValue: 'ap-northeast-2')
  }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Deploy') {
      steps {
        withAWS(credentials: 'aws-deploy', region: "${AWS_REGION}") {
          sh '''
            set -e
            aws --version
            test -d "${LOCAL_DIR}" || { echo "[ERROR] ${LOCAL_DIR} 폴더 없음"; exit 1; }

            echo "[DRY-RUN] S3 sync..."
            aws s3 sync "${LOCAL_DIR}/" "s3://${S3_BUCKET}/" --delete --dryrun

            echo "[DEPLOY] S3 sync..."
            aws s3 sync "${LOCAL_DIR}/" "s3://${S3_BUCKET}/" --delete

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
