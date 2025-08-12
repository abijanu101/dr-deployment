def kanikoPod = { dockerfilePath, imageName ->
  return """
    apiVersion: v1
    kind: Pod
    spec:
      restartPolicy: Never
      containers:
      - name: kaniko
        image: gcr.io/kaniko-project/executor:latest
        args:
        - --context=${env.WORKSPACE}
        - --dockerfile=${dockerfilePath}
        - --destination=abijanu101/${imageName}:latest
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker/config.json
          subPath: .dockerconfigjson
      volumes:
      - name: kaniko-secret
        secret:
          secretName: kaniko-secret
  """
}

pipeline {
  agent none
  stages {
    stage('Clone') {
      agent { kubernetes { yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
            - name: git
              image: bitnami/git
              command:
                - cat
              tty: true
      '''}}
      steps { checkout scm }
    }

    stage('Build React') {
      agent { kubernetes { yaml kanikoPod('frontend/Dockerfile', 'dr-react') } }
      steps { echo 'React build step reached' }
    }
    stage('Build Express') {
      agent { kubernetes { yaml kanikoPod('backend/Dockerfile', 'dr-express') } }
      steps { echo 'Express step reached' }
    }
    stage('Build SQL Init') {
      agent { kubernetes { yaml kanikoPod('db/Dockerfile', 'dr-sql-init') } }
      steps { echo 'SQL-init build step reached' }
    }

    stage('Deploy with Helm') {
      agent { kubernetes { yaml """
        apiVersion: v1
        kind: Pod
        spec:
          containers:
            - name: helm
              image: alpine/helm:3.14.0
              command:
                - cat
              tty: true
        """}}
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
