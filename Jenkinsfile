pipeline {
  parameters {
    string(name: 'tag', defaultValue: 'latest', description: 'tag {YYYYMMDD}{HHMMSS}')
  } 

  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: ubuntu
    command:
    - sleep
    args:
    - infinity
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
'''
       defaultContainer 'shell'
    }
  }

  environment {
    DOCKER_CREDENTIAL_ID = "wonkilee_dockerhub"
    K8S_CREDENTIAL_ID = "wonkilee_kubeconfig2"
  }

  stages {
      
    stage('Example') {
      steps {
        echo 'Hello World!'
      }
    }

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/chuirang/DevOps.git'
      }
    }
    
    stage('Docker build') {
      steps {
        container('docker') {
          script {
            env.IMAGE_TAG = "${params.tag}"
          }
          
          dir('docker') {
            withCredentials([usernamePassword(
              credentialsId: "${DOCKER_CREDENTIAL_ID}", // credentialsId
              usernameVariable: 'USERNAME', // 사용자명을 ${USERNAME} 환경변수에 mapping
              passwordVariable: 'PASSWORD'  // 사용자암호를 ${PASSWORD} 환경변수에 mapping
            )]) {
              //sh "docker login -u ${USERNAME} -p ${PASSWORD}"
              //sh "docker build -t ${USERNAME}/sampleapp:${env.IMAGE_TAG} ."
              //sh "docker push ${USERNAME}/sampleapp:${env.IMAGE_TAG}"
            }
          }
        }
      }
    }

    stage('Kubernetes deploy') {
      steps {
        kubernetesDeploy configs: "k8s/deployment.yaml", kubeconfigId: "${K8S_CREDENTIAL_ID}"
        //sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
        //sh 'chmod u+x ./kubectl'
        //sh "kubectl --kubeconfig=/root/.jenkins/.kube/config rollout restart deployment/sampleapp"
        withKubeConfig([credentialsId: "${K8S_CREDENTIAL_ID}"]) {
          sh 'kubectl  -f k8s/deployment.yaml rollout restart deployment/sampleapp'
        }
      }
    }
  }
}