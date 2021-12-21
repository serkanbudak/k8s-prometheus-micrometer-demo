
pipeline {

  environment {
      REGISTRY    = 'index.docker.io' // Configure your own registry
      REPOSITORY  = 'srknbdk'
      IMAGE       = 'hello'
    }
    agent {
        kubernetes {
            label 'kaniko'
            yaml """
kind: Pod
metadata:
  name: k8s-cd
spec:
  containers:
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true  
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: IfNotPresent
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: registry-credentials
          items:
            - key: .dockerconfigjson
              path: config.json
"""
        }
    }
    stages {
      
        stage('Build & Push Image') {
            environment {
                PATH        = "/busybox:$PATH"
            }
            steps {
                git 'https://github.com/dstar55/docker-hello-world-spring-boot.git'
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=${REGISTRY}/${REPOSITORY}/${IMAGE}:${BUILD_NUMBER}
                    '''
                }
            }
        }
        
        stage('Deploy') {
            steps {
                container(name: 'kubectl') {
                   
                  withKubeConfig([credentialsId: 'k8s-config']) {
                      sh 'echo $KUBECONFIG'
                      sh 'cat $KUBECONFIG'
                      sh 'kubectl get pods -A'
                      sh "echo ${REGISTRY}/${REPOSITORY}/${IMAGE}:${BUILD_NUMBER}"
                  }
                }
            }
        }
    }
}
