pipeline {
  agent any

  environment {
    IMAGE = "harbour.mohammadrifai.com/landing/landingpage:${BUILD_NUMBER}"
  }

  stages {
    stage('Clone') {
      steps {
        git branch: 'main', url: 'https://github.com/cutrocks/landingpage.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }

    stage('Push to Harbor') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'harbor-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo "$PASS" | docker login harbour.mohammadrifai.com -u "$USER" --password-stdin
            docker push $IMAGE
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: landingpage
  namespace: landingpage
spec:
  replicas: 3
  selector:
    matchLabels:
      app: landing
  template:
    metadata:
      labels:
        app: landing
    spec:
      containers:
      - name: landing
        image: $IMAGE
        ports:
        - containerPort: 80
      imagePullSecrets:
      - name: regcred
---
apiVersion: v1
kind: Service
metadata:
  name: landingpage-svc
  namespace: landingpage
spec:
  type: NodePort
  selector:
    app: landing
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
EOF
        '''
      }
    }
  }
}
