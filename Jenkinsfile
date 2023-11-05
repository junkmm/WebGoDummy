pipeline {
    agent {
        kubernetes {
            yaml """
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  - name: gitops
    image: bitnami/git:latest
    imagePullPolicy: Always
    command:
    - /bin/sh
    tty: true
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
        stage('Build Docker image') {
            environment {
                REPOSITORY  = 'kimhj4270'
                IMAGE       = 'godummyweb'
            }
            steps {
                container('kaniko') {
                    script {
                        sh "executor --dockerfile=Dockerfile --context=./ --destination=${REPOSITORY}/${IMAGE}:${GIT_COMMIT}"
                    }
                }
            }
        }

        stage('GitOps') {
            steps {
                container('gitops') {
                    sh """
                    git clone https://github.com/junkmm/GitopsDummy.git ./src
                    cd ./src
                    sed -i 's@kimhj4270/godummyweb:.*@kimhj4270/godummyweb:${GIT_COMMIT}@g' deploy.yaml
                    git add deploy.yaml
                    git commit -m "Update container image ${GIT_COMMIT}"
                    git push origin main
                    """
                }
            }
        }
    }
}