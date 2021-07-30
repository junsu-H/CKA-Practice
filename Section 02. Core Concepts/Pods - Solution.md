# Practice Test - Pods - Solution

# KEYWORD:
kubectl get
kubectl describe
kubectl edit

- pod 개수 확인

    ```bash
    kubectl get pods
    ```

- make nginx pod ( 방법 2개 )

    ```bash
    kubectl run nginx --image=nginx --generator=run-pod/v1
    ```

    [kubectl Usage Conventions](https://kubernetes.io/docs/reference/kubectl/conventions/)

    ```yaml
    # vim nginx-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx
    #    ports:
    #    - containerPort: 80
    #      protocol: TCP
    ```

- 다시 pod 개수 확인

    ```bash
    kubectl get pods --no-headers | wc -l
    ```

- new pod 이미지 확인

    ```bash
    kubectl describe newpods-j4wld | grep -i image
    ```

- new pod는 어느 노드에 있는가? ( 방법 2개 )

    ```bash
    kubectl get pods -o wide
    ```

    ```bash
    kubectl describe [pod-name]
    ```

- webapp 개수 확인

    ```bash
    kubectl get pods
    ```

- webapp pod 확인

    ```bash
    kubectl describe pod webapp
    ```

- webapp pod 삭제

    ```bash
    kubectl delete pod webapp
    ```

- redis-pod 생성

    ```yaml
    # nginx-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: redis-pod
    spec:
      containers:
      - name: redis
        image: redis123
    #    ports:
    #    - containerPort: 80
    #      protocol: TCP
    ```

    ```bash
    kubectl run redis --generator=run-pod/v1 --image=redis123
    ```

- redis-pod 이미지 이름 redis123→redis로 업데이트

    ```yaml
    # nginx-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: redis-pod
    spec:
      containers:
      - name: redis
        image: redis
    #    ports:
    #    - containerPort: 80
    #      protocol: TCP
    ```

    ```bash
    kubectl create -f redis-pod.yaml
    ```