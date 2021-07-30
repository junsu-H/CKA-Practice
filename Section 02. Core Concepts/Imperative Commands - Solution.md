# Imperative Commands - Solution

# KEYWORD:
kubectl expose

- 다음 조건에 맞는 pod를 생성하시오. yaml을 사용하지 않고 커맨드만 사용
    - Name: nginx-pod
    - Image: nginx:alpine

    ```bash
    kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:alpine
    ```

    [https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)

- 다음 조건에 맞는 pod를 생성하시오. yaml을 사용하지 않고 커맨드만 사용
    - Pod Name: redis
    - Image: redis:alpine
    - Labels: tier=db

    ```yaml
    kubectl run --generator=run-pod/v1 redis --image=redis:alpine --labels tier=db
    kubectl run --generator=run-pod/v1 redis --image=redis:alpine -l tier=db
    ```

- 위 redis pod에 service를 생성하시오. yaml을 사용하지 않고 커맨드만 사용
    - Service : redis-service
    - Port : 6379
    - Type : ClusterIp
    - selector : tier = db

    ```bash
    kubectl expose pod redis --name=redis-service --port=6379 --type=ClusterIP --selector=tier=db

    # 또는
    kubectl expose pod redis --name=redis-service --port=6379 --type=ClusterIP --labels=tier=db
    ```

    [Services](https://kubernetes.io/docs/tutorials/services/)

- 다음 조건에 맞는 deployment를 생성하시오. yaml을 사용하지 않고 커맨드만 사용
    - Name: webapp
    - Image: kodekloud/webapp-color
    - Replicas: 3

    ```bash
    kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
    kubectl scale deployment webapp --replicas=3
    ```