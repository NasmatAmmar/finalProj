pipeline {
  agent any

  environment {
    REGISTRY_URL = '352708296901.dkr.ecr.eu-west-1.amazonaws.com'
    ECR_REGION = 'eu-west-1'
    K8S_NAMESPACE = 'nasmat-hana-namespace'
    K8S_CLUSTER_NAME = 'devops-alfnar-k8s'
    K8S_CLUSTER_REGION = 'eu-north-1'
  }


  stages {
    stage('Creating NAMESPACE'){
      steps {
          sh '''
          kubectl create namespace nasmat-hana-namespace
          '''
      }
    }
    stage('MNIST Web Server - build'){
      when { branch "master" }
      steps {
          sh '''
         IMAGE="mnist-webserver-na-ha:0.0.${BUILD_NUMBER}"
            cd webserver
            aws ecr get-login-password --region $ECR_REGION | docker login --username AWS --password-stdin ${REGISTRY_URL}
            docker build -t ${IMAGE} .
            docker tag ${IMAGE} ${REGISTRY_URL}/${IMAGE}
            docker push ${REGISTRY_URL}/${IMAGE}
          '''
      }
    }

    stage('MNIST Web Server - deploy'){
        when { branch "master" }
        steps {
            sh '''
            cd infra/k8s
            IMG_NAME=mnist-webserver-na-ha:0.0.${BUILD_NUMBER}

            # replace registry url and image name placeholders in yaml
            sed -i "s/{{REGISTRY_URL}}/$REGISTRY_URL/g" mnist-webserver.yaml
            sed -i "s/{{K8S_NAMESPACE}}/$K8S_NAMESPACE/g" mnist-webserver.yaml
            sed -i "s/{{IMG_NAME}}/$IMG_NAME/g" mnist-webserver.yaml

            # get kubeconfig creds
            aws eks --region $K8S_CLUSTER_REGION update-kubeconfig --name $K8S_CLUSTER_NAME

            # apply to your namespace
            kubectl apply -f mnist-webserver.yaml -n $K8S_NAMESPACE
            '''
        }
    }


    stage('MNIST Predictor - build'){
        when { branch "master" }
        steps {
            sh '''
            IMAGE="mnist-predictor-na-ha:0.0.${BUILD_NUMBER}"
            cd ml_model
            aws ecr get-login-password --region $ECR_REGION | docker login --username AWS --password-stdin ${REGISTRY_URL}
            docker build -t ${IMAGE} .
            docker tag ${IMAGE} ${REGISTRY_URL}/${IMAGE}
            docker push ${REGISTRY_URL}/${IMAGE}
            '''
        }
    }
    stage('MNIST Predictor - deploy'){
        when { branch "master" }
        steps {
            sh '''
            cd infra/k8s
            IMG_NAME=mnist-predictor-na-ha:0.0.${BUILD_NUMBER}

            # replace registry url and image name placeholders in yaml
            sed -i "s/{{REGISTRY_URL}}/$REGISTRY_URL/g" mnist-predictor.yaml
            sed -i "s/{{K8S_NAMESPACE}}/$K8S_NAMESPACE/g" mnist-predictor.yaml
            sed -i "s/{{IMG_NAME}}/$IMG_NAME/g" mnist-predictor.yaml

            # get kubeconfig creds
            aws eks --region $K8S_CLUSTER_REGION update-kubeconfig --name $K8S_CLUSTER_NAME

            # apply to your namespace
            kubectl apply -f mnist-predictor.yaml -n $K8S_NAMESPACE
            '''
        }
    }
  }
}


