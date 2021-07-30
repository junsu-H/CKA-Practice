# Monitoring - Solution

# KEYWORD:
kubectl top node /
kubectl top pods

- git 파일 다운로드 받기

    ```bash
    git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
    ```

- 매트릭 서버를 배치

    ```bash
    # 모든 파일
    kubectl create -f .

    watch kubectl top nodes 
    ```

- node의 CPU 사용량, memory 사용량 보기

    ```bash
    kubectl top nodes

    # node01
    ```

- pod의 CPU 사용량, memory 사용량 보기

    ```bash
    kubectl top pods
    ```