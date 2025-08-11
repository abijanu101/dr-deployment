pipeline {
  agent none // Don't start a pod until we need one

  stages {

    stage('Build & Push Images') {
      agent {
        kubernetes {
          yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
                - name: kaniko
                  image: gcr.io/kaniko-project/executor:latest
                  volumeMounts:
                    - name: kaniko-secret
                      mountPath: /kaniko/.docker
              volumes:
                - name: kaniko-secret
                  secret:
                    secretName: kaniko-secret
          """
        }
      }
      steps {
        container('kaniko') {
          // Build & push React
          sh """
            /kaniko/executor \
              --context=git://github.com/abijanu101/dr-deployment.git \
              --dockerfile=frontend/Dockerfile \
              --destination=abijanu101/dr-react:latest
          """
          // Build & push Express
          sh """
            /kaniko/executor \
              --context=git://github.com/abijanu101/dr-deployment.git \
              --dockerfile=backend/Dockerfile \
              --destination=abijanu101/dr-express:latest
          """
          // Build & push SQL Init
          sh """
            /kaniko/executor \
              --context=git://github.com/abijanu101/dr-deployment.git \
              --dockerfile=db/Dockerfile \
              --destination=abijanu101/dr-sql-init:latest
          """
        }
      }
    }

    stage('Deploy with Helm') {
      agent {
        kubernetes {
          yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
                - name: helm
                  image: alpine/helm:3.14.0
                  command:
                    - cat
                  tty: true
          """
        }
      }
      steps {
        container('helm') {
          sh 'helm upgrade --install sql ./k8s/charts/base -f ./k8s/values/sql.yaml'
          sh 'helm upgrade --install react ./k8s/charts/base -f ./k8s/values/react.yaml'
          sh 'helm upgrade --install express ./k8s/charts/base -f ./k8s/values/express.yaml'
        }
      }
    }

  }
}
