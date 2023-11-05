// Kaniko에서 이미지 빌드 후 레지스트리에 Push하기 위해 권한이 필요하며, 아래 Secret을 생성 해주어야 한다.
// kubectl create secret -n {namespace} docker-registry registry-credentials \  
//    --docker-username=<username>  \
//    --docker-password=<password> \
//    --docker-email=<email-address>

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
			container('gitops'){
				environment{
					SHORT_COMMIT_HASH = env.SHORT_COMMIT_HASH
				}
				steps{
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
