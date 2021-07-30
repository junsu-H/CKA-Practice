# Environment Variables - Solution

# KEYWORD:
kubectl get cm /
kubectl create cm -h /
envFrom

- 디폴트 네임스페이스에 존재하는 pod 보기

    ```bash
    kubectl get pods -n default
    ```

- webapp-color pod에 환경 변수의 이름 보기

    ```bash
    kubectl describe pods webapp-color | grep -i -A5 -B5 env

    # 또는
    kubectl exec -it webapp-color -- env

    # APP_COLOR: pink
    # APP_COLOR가 답
    ```

- webapp-color pod에 환경 변수의 값 보기

    ```bash
    kubectl describe pods webapp-color | grep -i -A5 -B5 env

    # 또는
    kubectl exec -it webapp-color -- env

    # APP_COLOR: pink
    # pink가 답
    ```

- 다음 조건을 만족하는 pod를 생성하시오
    - Pod Name: webapp-color
    - Label Name: webapp-color
    - Env: APP_COLOR=green

    ```yaml
    # vim webapp-color.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp-color
    #  labels:
    #    name: webapp-color
    spec:
      containers:
      - name: webapp-color
        image: kodekloud/webapp-color
        env:
        - name: APP_COLOR
          value: green
    #  nodeName: node01
    ```

    ```bash
    kubectl get pods
    kubectl delete pods webapp-color
    kubectl create -f webapp-color.yaml
    ```

    ```bash
    k run --generator=run-pod/v1 webapp-color \
        --image=kodekloud/webapp-color \
        --env=APP_COLOR=green
    ```

- configmap은 몇 개가 있는가?

    ```bash
    # 둘 다 가능
    kubectl get cm
    kubectl get configmaps
    ```

- db-config에서 DB_HOST는?

    ```bash
    kubectl describe cm db-config # 얘로 보면 좀 이상하게 나옴
    kubectl edit cm db-config # 얘로 보기
    ```

- 다음 조건을 만족하는 configmap을 작성하여라
    - ConfigName Name: webapp-config-map
    - Data: APP_COLOR=darkblue

    ```bash
    kubectl create configmap -h
    kubectl create configmap webapp-config-map --from-literal APP_COLOR=darkblue
    ```

    ```yaml
    # webapp-config-map.yaml

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: webapp-config-map
    data:
      APP_COLOR: darkblue
    ```

    [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

- 위에서 만든 configmap을 사용해서 pod env를 수정하여라
    - Pod Name: webapp-color
    - EnvFrom: webapp-config-map

    ```bash
    kubectl describe pods webapp-color | grep -i image
    # kodekloud/webapp-color

    kubectl delete pods webapp-color
    vim webapp-color-pod.yaml
    ```

    ```yaml
    # webapp-color-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp-color
    spec:
      containers:
      - name: webapp-color
        image: kodekloud/webapp-color

    #   아래를 다음과 같이 수정
    #   env:
    #   - name: APP_COLOR
    #     value: green

        envFrom:
        - configMapRef:
            name: webapp-config-map
    ```

    ```bash
    kubectl create -f webapp-color-pod.yaml
    ```

    [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)