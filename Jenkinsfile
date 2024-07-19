pipeline {
  agent any
  environment {
      branch_name="${Branch}"
      release_type="${Release}"
      AWS_ACCESS_KEY = credentials('7a5f353a-4c21-491f-a4a6-cb4738f5b0a9') 
      AWS_SECRET_KEY = credentials('590a8e9b-8854-431e-817c-08faef36799d')
      AWS_REGION = credentials('efcaf984-bf4c-4f34-a84c-5f0438bfcdba')
      AWS_ACCOUNT_ID = credentials('c620055a-b75b-40b2-a390-d780f977faa8')
  }

  stages {
    stage('Git Clone') {
        script {
            git(url: 'https://git.assistanz.com/stackbill/sb-helm-charts.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: 'master')
            sh 'ls -al'
        }
    }
  }
}