pipeline {
  agent any

  environment {
    AWS_REGION        = "ap-southeast-2"
    AWS_ACCOUNT_ID    = "751269371912"
    ECR_REPO          = "tourism-app"
    EKS_CLUSTER_NAME  = "tourism-cluster"
    IMAGE_URI         = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/meghanachepuri/Project-poc.git'
      }
    }

    stage('Login to ECR') {
      steps {
        withCredentials([
          string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
          string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
          sh '''
            aws ecr get-login-password --region $AWS_REGION \
              | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
          '''
        }
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        sh '''
          docker build -t $ECR_REPO .
          docker tag $ECR_REPO:latest $IMAGE_URI
          docker push $IMAGE_URI
        '''
      }
    }

    stage('Update kubeconfig') {
      steps {
        withCredentials([
          string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
          string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
          sh 'aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION'
        }
      }
    }

    stage('Deploy to EKS') {
      steps {
        withCredentials([
          string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
          string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
          sh '''
            kubectl set image deployment/tourism-deployment \
              tourism-container=$IMAGE_URI || \
              kubectl apply -f deployment.yaml
            kubectl apply -f service.yaml
          '''
        }
      }
    }
  }
}

