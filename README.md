### k8s-node-app.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-dep
  labels:
    app: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: demo
        image: devopshint/node-app:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: demo-k8s-service
spec:
  selector:
    app: demo
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

`.github/workflows/deploy-to-minikube-github-actions.yaml`
```yaml
on:
    push:
      branches:
        - main
jobs:
    job1:
      runs-on: ubuntu-latest
      name: Build Node.js Docker Image and deploy to Minikube
      steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Start minikube
        uses: medyagh/setup-minikube@master
      - name: Try the cluster!
        run: kubectl get pods -A
      - name: Build image
        run: |
            export SHELL=/bin/bash
            eval $(minikube -p minikube docker-env)
            docker build -f ./Dockerfile -t devopshint/node-app:latest .
            echo -n "verifying images:"
            docker images         
      - name: Deploy to Minikube
        run:
          kubectl apply -f k8s-node-app.yaml
      - name: Wait for the pod to be ready
        run: kubectl wait --for=condition=ready pod -l app=demo --timeout=60s
      - name: Test service URLs
        run: |
            minikube service list
            minikube service demo-k8s-service --url
```

![image](https://github.com/mounishvatti/minikube/assets/76279858/4e115bde-7834-436c-ac02-ff85291e81fc)
