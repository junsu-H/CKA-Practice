# Init Containers - Solution

# KEYWORD:
initContainers

- 현재 pod에서 init containers를 사용하는 pod는?

    ```bash
    kubectl get pods
    kubectl describe pods blue | grep -i init # Init Containers 존재
    kubectl describe pods green | grep -i init
    kubectl describe pods red | grep -i init
    ```

- init containers에 사용된 images는?

    ```bash
    kubectl describe pods blue | grep -i -A10 init

    # busybox
    ```

- init containers의 상태는?

    ```bash
    kubectl describe pod blue | grep -i -A10 -B10 state

    # Terminated
    ```

- init containers가 종료된 이유는?

    ```bash
    kubectl describe pods blue | grep -i -A5 -B5 reason
    ```

- pod purple에 init containers가 몇 개가 있는가?

    ```bash
    kubectl get pods
    kubectl describe pods purple | grep -i -A10 init

    # 2개
    ```

- pod purple에 init containers가 사용하는 이미지는 몇 개가 있는가?

    ```bash
    kubectl describe pods purple
    ```

- pod purple의 상태는?

    ```bash
    kubectl describe pod purple | grep -i -B10 state

    # purple-container:
    #   state 봐야 됨
    # Waiting
    ```

- 시간이 얼마나 흐른 뒤에 pod purple을 사용할 수 있나?

    ```bash
    kubectl describe pod purple | grep -i sleep

    # init containers1은 sleep 600
    # init containers2은 sleep 1200
    # 따라서 600+1200/sec는 30분
    ```

- pod red 업데이트 하기
    - Pod: red
    - initContainer Configured Correctly
    - sleep 20

    ```bash
    kubectl get pods -o yaml > red-pod.yaml
    ```

    ```yaml
    # red-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: red  
      namespace: default
    spec:
      nodeName: node01
      containers:  
      - name: red-container
        image: busybox:1.28
        command: ["sh", "-c", "echo The app is running! && sleep 3600"]

    #    위와 동일
    #    command:
    #    - sh
    #    - -c
    #    - echo The app is running! && sleep 3600
      
      initContainers:
      - name: init-myservice
        image: busybox
        command: ["sleep", "20"]
    ```

    ```bash
    kubectl delete pods red
    kubectl create -f red-pod.yaml
    ```

- Init:CrashLoopBackOff 걸리는 pod oragne 트러블 슈팅

    ```bash
    # cf) Init:CrashLoopBackOff 초기화 컨테이너가 반복적으로 실패

    kubectl get pods
    kubectl describe pods orange
    kubectl get pods orange -o yaml > orange-pod.yaml
    vim orange-pod.yaml
    ```

    ```yaml
    # orange-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: orange
    spec:
      containers: 
      - name: orange-container
        image: busybox:1.28
        command:
        - sh
        - -c
        - echo The app is running! && sleep 3600

      initContainers:
      - name: init-myservice
        image: busybox 
        command: ["sh", "-c", "sleep 2"] # sleeeep 2에서 수정
        nodeName: node01
    ```

    ```bash
    kubectl delete pods oragne
    kubectl create -f orange-pod.yaml
    ```