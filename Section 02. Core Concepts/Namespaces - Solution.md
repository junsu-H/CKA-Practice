# Namespaces - Solution

# KEYWORD:
kubectl get ns

- namespace 개수

    ```bash
    kubectl get ns --no-headers | wc -l
    ```

- namespace 'research'에 있는 Pod 개수

    ```bash
    kubectl get pods -n research
    kubectl get pods --namespace=research
    ```

- namespace가 'finance'인 Pod 생성

    ```yaml
    # redis-pod.yaml
    apiVersion: v1kind: Pod
    metadata:  
      name: redis
      namespace: finance
    spec:  
      containers:
      - name: redis-con
        image: redis

    kubectl create -f redis-pod.yaml

    # yaml 생성 없이 해보기
    ```

    ```bash
    kubectl run --generator=run-pod/v1 redis --image=redis -n finance --dry-run
    ```

- 'blue' Pod가 있는 namespace

    ```bash
    kubectl get pods --all-namespace | grep blue
    ```

![Untitled 1](https://user-images.githubusercontent.com/63388678/104253489-e99c1200-54b7-11eb-858b-c676c7587c23.png)

- Blue 애플리케이션은 'marketing' 네임 스페이스에서 데이터베이스 'db-service'에 액세스하기 위해 사용해야하는 DNS 이름

    ```bash
    # [svc name] + [ns] + "svc" + [domain]

    # svc name (db-service)
    kubectl get svc -n marketing

    # ns (marketing)
    kubectl get ns

    # domain (cluster.loacl)
    kubectl get configmap --all-namespaces
    kubectl describe configmap coredns -n kube-system | grep -i kube
    # kubernetes cluster.local

    # 가능
    # db-service
    # db-service.marketing
    # db-service.marketing.svc
    # db-service.marketing.svc.cluster.local
    ```

    ![Untitled 2](https://user-images.githubusercontent.com/63388678/104253490-ea34a880-54b7-11eb-805c-ad7b2f509feb.png)

- Blue 애플리케이션이 'dev' 네임 스페이스의 데이터베이스 'db-service'에 액세스하기 위해 사용해야하는 DNS 이름

    ```bash
    # [svc name] + [ns] + "svc" + [domain]

    # svc name (db-service)
    kubectl get svc -n dev

    # ns (dev)
    kubectl get ns

    # domain (cluster.loacl)
    kubectl get configmap --all-namespaces
    kubectl describe configmap coredns -n kube-system | grep -i kubernetes
    # kubernetes cluster.local

    # 가능
    # db-service
    # db-service.dev
    # db-service.dev.svc
    # db-service.dev.svc.cluster.local
    ```

    ![Untitled 3](https://user-images.githubusercontent.com/63388678/104253491-eb65d580-54b7-11eb-9e76-c01c291b4b98.png)