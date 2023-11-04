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
            """
        }
    }

    stages {
         stage('My test') {
            steps {
                container('docker') {
                    sh "echo hello world"
                    sh "docker ps"
                }
            }
        }
    }
}
