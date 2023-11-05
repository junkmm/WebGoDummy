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
                script {
                    withCredentials([usernamePassword(credentialsId: 'git_cre', passwordVariable: 'password', usernameVariable: 'username')]) {
                        container('gitops') {
                            git credentialsId: 'git_cre', url: 'https://github.com/junkmm/GitopsDummy.git', branch: 'main'
                            sh """
                            git init
                            git config --global --add safe.directory /home/jenkins/agent/workspace/demo2
                            git config --global user.email 'jenkins@jenkins.com'
                            git config --global user.name 'jenkins'
                            git add deploy.yaml
                            git commit -m 'Update: Image ${GIT_COMMIT}'
                            git remote set-url origin https://${username}:${password}@github.com/junkmm/GitopsDummy.git
                            git push origin main
                            """
                        }
                    }
                }
            }
        }
    }
}