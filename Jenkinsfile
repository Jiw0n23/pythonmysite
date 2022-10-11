REGION = 'ap-northeast-2'
EKS_API = 'https://AE0727C3A0B3F4E50B61887B9F08B9F1.gr7.ap-northeast-2.eks.amazonaws.com'
EKS_CLUSTER_NAME='project-test'
EKS_NAMESPACE='default'
EKS_JENKINS_CREDENTIAL_ID='kubectl-deploy-credentials'
ECR_PATH = '678481348986.dkr.ecr.ap-northeast-2.amazonaws.com'
ECR_IMAGE = 'projectrp'
AWS_CREDENTIAL_ID = 'jenkins-aws-anderson-credentials'

pipeline {
  agent {
    node {        
      stage('Clone Repository'){
        checkout scm
    } 
      stage('Docker Build'){
        // Docker Build
        docker.withRegistry("https://${ECR_PATH}", "ecr:${REGION}:${AWS_CREDENTIAL_ID}"){
            image = docker.build("${ECR_PATH}/${ECR_IMAGE}", "--network=host --no-cache .")
        }
      post {
        success {
          slackSend (
            channel: project2,
            color: SLACK_SUCCESS_COLOR,
            message: "Docker Image Build에 성공하였습니다."
          )
        } 
        failure {
          slackSend (
            channel: project2,
            color: SLACK_FAIL_COLOR,
            message: "Docker Image Build에 실패하였습니다."
          )
        }
   }
}
      stage('Push to ECR'){
        docker.withRegistry("https://${ECR_PATH}", "ecr:${REGION}:${AWS_CREDENTIAL_ID}"){
            image.push("v${env.BUILD_NUMBER}")
          }
      }
      stage('CleanUp Images'){
        sh"""
          docker rmi ${ECR_PATH}/${ECR_IMAGE}:v$BUILD_NUMBER
          docker rmi ${ECR_PATH}/${ECR_IMAGE}:latest
          """
    }
      stage('Deploy to K8S'){
        withKubeConfig([credentialsId: "kubectl-deploy-credentials",
                        serverUrl: "${EKS_API}",
                        clusterName: "${EKS_CLUSTER_NAME}"]){
            sh "sed 's/IMAGE_VERSION/${env.BUILD_ID}/g' deploy.yml > output.yml"
            sh "aws eks --region ${REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME}"
            sh "kubectl apply -f output.yml"
            sh "rm output.yml"
        }
    }
}
}
}
