# Labels and Selectors - Solution

# KEYWORD:
kubectl get all --show-labels /
kubectl get pods -l

- labels이 env=dev인 것

    ```bash
    kubectl get pods -l env=dev --no-headers | wc -l

    kubectl get pod --selector env=dev --no-headers | wc -l

    # wc 명령어는 파일 내의 라인, 단어 문자의 수를 출력

    # kubectl get pod --selector tier
    # kubectl get pod --selector bu
    ```

- labels이 bu=finance인 것

    ```bash
    kubectl get pods -l bu=finacne --no-headers | wc -l
    kubectl get pods --selector bu=finance --no-headers | wc -l
    ```

- POD, ReplicaSet 및 기타 객체를 포함하여 env=prod에 몇 개의 객체가 있습니까?

    ```bash
    kubectl get all -l env=prod --no-headers | wc -l
    ```

- labels이 env=prod이고, bu=finance, tier=frontend인 것

    ```bash
    # 문제가 이해가 안 되면
    kubectl get all --show-labels

    kubectl get pods -l env=prod,bu=finance,tier=frontend
    kubectl get pods --selector env=prod,bu=finance,tier=frontend

    # 띄어쓰기하면 안 됨.
    kubectl get pods --selector env=prod, bu=finance, tier=frontend
    ```

- replicaset-definition-1.yaml 작성

    ```bash
    kubectl create -f replicaset-definition-1.yaml

    # err output
    > The ReplicaSet "replicaset-1" is invalid: spec.template.metadata.labels: Invalid
    value: map[string]string{"tier":"nginx"}: `selector` does not match template `labdon't print headers (default print headers).
    els`

    vim replicaset-definition-1.yaml

    ```

    ```yaml
    # replicaset-definition-1.yaml

    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: replicaset-1
    spec:
      replicas: 2
      selector:
        # 이 부분은 rs가 갖는 matchlabels
        matchLabels:
          # tier: nginx를 아래와 같이 수정
          tier: frontend
      template:
        metadata:
          # 여기는 pod가 갖는 labels
          labels:
            tier: frontend
        spec:
          containers:
          - name: nginx
            image: nginx
    ```

    ```bash
    kubectl create -f replicaset-definition-1.yaml 
    ```