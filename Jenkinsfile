pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
spec:
  dnsPolicy: Default
  containers:
    - name: docker
      image: docker:latest
      command:
        - cat
      tty: true
      privileged: true
      volumeMounts:
        - name: dockersock
          mountPath: /var/run/docker.sock
  volumes:
    - name: dockersock
      hostPath:
        path: /var/run/docker.sock
            """
        }
    }

    stages {
         stage('Build Docker image') {
            steps {
                container('docker') {
					sh 'docker image ls'
					sh 'docker build -t kimhj4270/godummyweb:latest .'
					sh 'docker image ls'
                }
            }
        }
    }
}
