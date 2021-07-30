# Practice Test - Deployments - Solution

# KEYWORD:
kubectl get deploy

- pod 개수

    ```bash
    kubectl get pods
    ```

- replicaset 개수

    ```bash
    kubectl get rs
    ```

- Deployment 개수

    ```bash
    kubectl get deploy
    ```

- Deployment image 오류

    ```bash
    kubectl describe deploy frontend-deployment

    # Pod image trouble shooting
    kubectl get pods -o wide
    ```

- deployment-definition-1.yaml 파일을 사용하여 Deployment 만들기

    ```bash
    kubectl get depoly

    vim deployment-definition-1.yaml 
    > **kind: Deployment
    ****
    kubectl apply -f deployment-definition-1.yaml
    ```

- deployment-definition-2.yaml 파일을 사용하여 Deployment 만들기

    ```yaml
    # deployment-definition-2.yaml
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: httpd-frontend  # httpd-frontend-deploy
    spec:
      replicas: 3
      selector:
        matchLabels:
            name: httpd-frontend-pod-labels
      template:
        metadata:
            labels: httpd-frontend-pod-labels
            name: httpd-frontend-pod
        spec:
          containers:
          - name: httpd-frontend-con
            image: httpd:2.4-alpine
    ```