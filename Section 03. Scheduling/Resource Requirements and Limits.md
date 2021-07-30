# Resource Requirements and Limits - Solution

# KEYWORD:
request /
limit

- pod rabbit의 CPU requests는?

    ```bash
    kubectl describe pod rabbit | grep -A5 -i requests

    # cpu: 1
    ```

    [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

- pod rabbit 삭제

    ```bash
    kubectl delete pods rabbit
    ```

- pod elephant의 상태

    ```bash
    kubectl get pods elephant
    ```

- pod elephant의 limit memory는?

    ```bash
    kubectl describe pods elephant | grep -i -A5 limit

    # 10Mi 
    ```

- pod elephant는 15Mi의 메모리를 소비하므로 20Mi로 수정하시오.

    ```bash
    vim elephant-pod.yaml
    ```

    ```yaml
    # elephant-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: elephant
    spec:
      containers:
      - name: elepahant-con
        image: polinux/stress
        resources:
          requests:
            memory: "15Mi"
          limits:
            memory: "20Mi"
    ```

    ```bash
    kubectl create -f elephant-pod.yaml
    ```

    [Managing Compute Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)

- pod elephant 상태 확인

    ```bash
    kubectl get pods elephant
    ```

- pod elephant 삭제

    ```bash
    kubectl delete pods elephant
    ```