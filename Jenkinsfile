
pipeline {

  environment {
      REGISTRY    = 'index.docker.io' // Configure your own registry
      REPOSITORY  = 'srknbdk'
      IMAGE       = 'example-app'
      GIT_HASH    = GIT_COMMIT.take(7)
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
  - name: k8s-helm
    image: lachlanevenson/k8s-helm
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
                checkout scm
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=${REGISTRY}/${REPOSITORY}/${IMAGE}:${GIT_HASH}
                    '''
                }
            }
        }
        
        stage('Deployment k8s') {
            steps {
                checkout scm
                container(name: 'k8s-helm') {
                   
                  withKubeConfig([credentialsId: 'k8s-config']) {
                      sh 'echo $KUBECONFIG'
                      sh 'cat $KUBECONFIG'
                      sh "helm upgrade --install example-app --set image.tag=${GIT_HASH} -n k8s-prometheus-micrometer-demo ./helm/example-app"
                      sh 'helm list -A'
                      
                  }
                }
            }
        }
    }
}
