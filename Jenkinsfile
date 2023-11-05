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
                        def fullCommitHash = env.GIT_COMMIT
                        def shortCommitHash = fullCommitHash.take(7)
                        sh "executor --dockerfile=Dockerfile --context=./ --destination=${REPOSITORY}/${IMAGE}:${shortCommitHash}"
                        env.SHORT_COMMIT_HASH = shortCommitHash
                    }
                }
            }
        }

        stage('GitOps') {
            environment {
                SHORT_COMMIT_HASH = env.SHORT_COMMIT_HASH
            }
            steps {
                container('gitops') {
                    sh """
                    git clone https://github.com/junkmm/GitopsDummy.git ./src
                    cd ./src
                    sed -i 's@kimhj4270/godummyweb:.*@kimhj4270/godummyweb:${SHORT_COMMIT_HASH}@g' deploy.yaml
                    git add deploy.yaml
                    git commit -m "Update container image ${SHORT_COMMIT_HASH}"
                    git push origin main
                    """
                }
            }
        }
    }
}