# Secrets - Solution

# KEYWORD:
kubectl get secrets /
kubectl create secret /
echo -n "root" | base64

- 현재 네임스페이스에서 secret 개수

    ```bash
    kubectl get secrets -n default
    ```

- default-token-zwk8k에 있는 secret 개수

    ```bash
    kubectl get secrets | grep -i -A5 -B5 data

    # 3개
    ```

- default-token-zwk8k의 type

    ```bash
    kubectl describe secrets | grep -i type
    ```

- default-token에서 secret data가 아닌 것은?

    ```bash
    kubectl describe secret default-token-l6sg5

    # 바이트나 해시로 표현되지 않은 값
    # type이 답
    ```

![Untitled 1](https://user-images.githubusercontent.com/63388678/104115004-fbfa3c80-534d-11eb-99ae-9709646710d9.png)

- mysql에 secret 적용하기
    - Secret Name: db-secret
    - Secret 1: DB_Host=sql01
    - Secret 2: DB_User=root
    - Secret 3: DB_Password=password123

    ```bash
    # create로 만들 땐, value를 base64로 인코딩 안 해도 된다.

    kubectl create secret -h
    kubectl create secret generic -h

    kubectl create secret generic db-secret \
        --from-literal=DB_Host=sql01 \
        --from-literal=DB_User=root \
        --from-literal=DB_Password=password123
    ```

    ---

    ```bash
    # yaml로 만들 땐 반드시 base64로 인코딩해야 한다.

    echo -n "sql01" | base64
    echo -n "root" | base64
    echo -n "password123" | base64
    # echo -n은 개행 제외한 문자, -n 안 주면 sql01\n이 인코딩됨.
    ```

    ```yaml
    # vim mysql-secret.yaml

    apiVersion: v1
    kind: Secret
    metadata:
      name: db-secret
    data:
      DB_Host: c3FsMDE=
      DB_User: cm9vdA==
      DB_Password: cGFzc3dvcmQxMjM=

    # echo -n "sql01" | base64
    # echo -n "root" | base64
    # echo -n "password123" | base64

    # data:
    #   DB_Host=sql01
    #   DB_User=root
    #   DB_Password=password123
    ```

    ```bash
    kubectl create -f mysql-secret.yaml
    ```

    [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

- webapp에 secret 적용
    - Pod name: webapp-pod
    - Image name: kodekloud/simple-webapp-mysql
    - Env From: Secret=db-secret

    ```bash
    kubectl get pods
    kubectl delete pods webapp-pod
    vim webapp-pod.yaml
    ```

    ```yaml
    # webapp-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp-pod
    spec:
      containers:
      - name: webapp-con
        image: kodekloud/simple-webapp-mysql
        envFrom:    
        - secretRef:        
            name: db-secret
    ```

    ```bash
    kubectl create -f webapp-pod.yaml
    ```

    [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)