pipeline {
    agent {
        kubernetes {
            label 'kaniko'
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
					}
				}
			}
		}
	}
}
