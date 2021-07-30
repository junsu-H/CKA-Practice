# Commands and Arguments - Solution

# KEYWORD:
dockerfile은 yaml에 무시된다. /
.spec.containers.command /
.spec.containers.args /
command는 디폴트값 /
args는 옵션값

- default namespace에서 pod 개수

    ```bash
    kubectl get pods
    ```

- ubuntu-sleeper에 command는?

    ```bash
    kubectl describe pods ubuntu-sleeper | grep -i -A5 -B5 command
    ```

- ubuntu-sleeper-2.yaml를 사용해서 sleep 5000 이미지 제작

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: ubuntu-sleeper-2
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["sleep", "5000"]
    ```

    ```bash
    kubectl create -f ubuntu-sleeper-2.yaml
    ```

- ubuntu-sleeper-3.yaml를 사용해서 sleep 1200 이미지 제작

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: ubuntu-sleeper-3
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["sleep", "1200"]
    ```

    ```bash
    kubectl create -f ubuntu-sleeper-3.yaml
    ```

- ubuntu-sleeper-3.yaml를 사용해서 sleep 2000으로 수정

    ```bash
    kubectl get pods
    kubectl delete pods ubuntu-sleeper-3
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata: 
      name: ubuntu-sleeper-3
    spec:
      containers:
      - name: ubuntu
        image: ubuntu
        command:
        - "sleep"
        - "2000"
    ```

    ```bash
    kubectl create -f ubuntu-sleeper-3.yaml
    ```

- Dockerfile을 통해 시작시 어떤 명령이 실행되는지 확인하시오.

    ```bash
    cd webapp-color
    vim Dockerfile
    ```

    ```bash
    # Dockerfile
    FROM python:3.6-alpine

    RUN pip install flask

    COPY . /opt/

    EXPOSE 8080

    WORKDIR /opt

    ENTRYPOINT ["python", "app.py"]

    # python app.py
    ```

- Dockerfile2을 통해 시작시 어떤 명령이 실행되는지 확인하시오.

    ```bash
    cd webapp-color
    vim Dockerfile
    ```

    ```bash
    # Dockerfile2

    FROM python:3.6-alpine

    RUN pip install flask

    COPY . /opt/

    EXPOSE 8080

    WORKDIR /opt

    ENTRYPOINT ["python", "app.py"]

    CMD ["--color", "red"]

    # python app.py --color red
    ```

- Dockerfile2와 webapp-color-pod.yaml을 통해 명령어 확인

    ```bash
    # Dockerfile은 무시된다.

    cd webapp-color-2
    vim Dockerfile2
    ```

    ```bash
    # Dockerfile2

    FROM python:3.6-alpine

    RUN pip install flask

    COPY . /opt/

    EXPOSE 8080

    WORKDIR /opt

    ENTRYPOINT ["python", "app.py"]

    CMD ["--color", "red"]

    # python app.py --color red
    ```

    ```bash
    vim webapp-color-pod.yaml
    ```

    ```yaml
    # webapp-color-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp-green
      labels:
          name: webapp-green
    spec:
      containers:
      - name: simple-webapp
        image: kodekloud/webapp-color
        command: ["--color","green"]

    ---
    # 다음과 같이 수정됨
    python app.py --color red ---> --color green
    ```

- Dockerfile2와 webapp-color-pod-2.yaml을 통해 명령어 확인

    ```bash
    # Dockerfile은 무시된다.

    cd webapp-color-3
    vim Dockerfile2
    ```

    ```bash
    # Dockerfile2

    FROM python:3.6-alpine

    RUN pip install flask

    COPY . /opt/

    EXPOSE 8080

    WORKDIR /opt

    ENTRYPOINT ["python", "app.py"]

    CMD ["--color", "red"]

    # python app.py --color red
    ```

    ```bash
    vim webapp-color-pod-2.yaml
    ```

    ```yaml
    # webapp-color-pod-2.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp-green
      labels:
          name: webapp-green
    spec:
      containers:
      - name: simple-webapp
        image: kodekloud/webapp-color
        command: ["python", "app.py"]
        args: ["--color", "pink"]

    ---
    # 다음과 같이 수정됨
    python app.py --color red ---> python app.py --color pink
    ```

- 기본 블루, 이후에 그린으로 표시하시오.
    - Pod Name: webapp-green
    - Image: kodekloud/webapp-color
    - Command line arguments: --color=green

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp-green
    spec:
      containers:
      - name:  webapp-green-con
        image: kodekloud/webapp-color
        commnad: ["--color", "blue"]
        args: ["--color", "green"]
    ```