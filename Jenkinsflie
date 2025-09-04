pipeline {
  agent any
  environment {
    AWS_REGION='ap-northeast-2'
    S3_BUCKET='hsj-project2'
    CF_DIST_ID='E6S76RL7FIVT2'
    SRC_DIR='web'
  }
  stages {
    stage('Checkout'){ steps { checkout scm; sh 'ls -al' } }
    stage('AWS CLI'){ steps { sh '''
      set -e
      if ! command -v aws >/dev/null; then
        ARCH=$(uname -m); TMP=/tmp/awscli; mkdir -p $TMP
        curl -sL "https://awscli.amazonaws.com/awscli-exe-linux-${ARCH}.zip" -o "$TMP/awscliv2.zip"
        unzip -q "$TMP/awscliv2.zip" -d $TMP; sudo $TMP/aws/install || sudo $TMP/aws/install --update
      fi
      aws --version
    ''' } }
    stage('Deploy'){ steps {
      withCredentials([usernamePassword(credentialsId: 'aws-deploy', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        sh '''
          set -e
          aws s3 sync "${SRC_DIR}/" "s3://${S3_BUCKET}/" --delete
          aws cloudfront create-invalidation --distribution-id "${CF_DIST_ID}" --paths "/*"
        '''
      }
    } }
  }
}
