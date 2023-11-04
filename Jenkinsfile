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
						sh "executor --dockerfile=Dockerfile --context=./ --destination=${REPOSITORY}/${IMAGE}:latest --destination=${REPOSITORY}/${IMAGE}:${GIT_COMMIT}"
					}
				}
			}
		}
	}
}
