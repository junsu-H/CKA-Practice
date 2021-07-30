# Multi Container PODs - Solution

# KEYWORD:
kubectl exec -it [pod] -c [container] -- bash

- pod에 생성된 컨테이너 개수

    ```bash
    kubectl get pods
    ```

- blue pod에 사용된 컨테이너 이미지

    ```bash
    kubectl describe pods blue | grep -i event -A10
    ```

- 다음 pod를 생성하시오.
    - Name: yellow
    - Container 1 Name: lemon
    - Container 1 Image: busybox
    - Container 2 Name: gold
    - Container 2 Image: redis

    ```yaml
    # yellow-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: yellow
    spec:
      containers:
      - name: lemon
        image: busybox

      - name: gold
        image: redis
    ```

    ```bash
    kubectl create -f yellow.yaml
    ```

- 네임스페이스가 elastic-stack인 app pod 점검

    ```bash
    kubectl get pods -n elastic-stack
    kubectl get pods app -n elastic-stack
    ```

- 네임스페이스가 elastic-stack인 app pod 점검

    ```bash
    kubectl describe pods app -n elastic-stack | grep -i -A10 event
    ```

- app 파드에서 로그인에 문제 있는 user 확인

    ```bash
    kubectl exec -it app -n elastic-stack -- cat /log/app.log
    kubectl exec -it app -n elastic-stack -- cat /log/app.log | grep -i warning
    # USER5
    ```

- pod에 조건에 맞게 컨테이너를 추가만 하시오.
    - Name: app
    - Container Name: sidecar
    - Container Image: kodekloud/filebeat-configured
    - Volume Mount: log-volume
    - Mount Path: /var/log/event-simulator/
    - Existing Container Name: app
    - Existing Container Image: kodekloud/event-simulator

    ```bash
    kubectl get pods app -n elastic-stack -o yaml > app-pod.yaml
    vim app-pod.yaml
    ```

    ```yaml
    # app-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: app
      namespace: elastic-stack
      labels:
        name: app
    spec:
      containers:
      - name: app
        image: kodekloud/event-simulator
        volumeMounts:
        - mountPath: /log
          name: log-volume

      # 아래만 추가하면 됨.
      - name: sidecar
        image: kodekloud/filebeat-configured
        volumeMounts:
        - mountPath: /var/log/event-simulator/
          name: log-volume

      volumes:
      - name: log-volume
      # default 값임.
      # emptyDir: {}
    ```

    ```bash
    kubectl delete pods app -n elastic-stack
    kubectl create -f app-pod.yaml
    ```

    [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

- kibana에서 로그 보기

    [Kubernetes CKAD - Kibana Dashboard](https://www.loom.com/share/c2ae70197e8340a0ba77fc1de8179182)