# Security Contexts - Solution

# KEYWORD:

```yaml
spec:
  containers:
  - securityContext:
       capabilities:
         add: ["SYS_TIME"]
```

- 'ubuntu-sleeper'포드 내에서 휴면 프로세스를 실행하는 데 사용자는 무엇입니까? 현재 (기본) 네임 스페이스에서

    ```bash
    kubectl get pods
    kubectl exec ubuntu-sleeper -- whoami

    # root
    ```

- 사용자 ID 1010으로 절전 프로세스를 실행하려면 'ubuntu-sleeper'포드를 편집하십시오. 
참고 : 필요한 사항만 변경하십시오. 포드의 이름이나 이미지를 수정하지 마십시오.
    - Pod Name: ubuntu-sleeper
    - Image Name: ubuntu
    - SecurityContext: User 1010

    ```bash
    kubectl get pods ubuntu-sleeper -o yaml > ubuntu-sleeper.yaml
    ```

    ```yaml
    # ubuntu-sleeper.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: ubuntu-sleeper
    spec:
      securityContext:
        runAsUser: 1000
      containers:
      -  image: ubuntu
         name: ubuntu
         command: ["sleep", "4800"]
         securityContext:
           runAsUser: 1010
    ```

    ```bash
    kubectl delete pods ubuntu-sleeper
    kubectl create -f ubuntu-sleeper.yaml
    ```

    [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

- 'multi-pod.yaml'이라는 포드 정의 파일이 제공됩니다. '웹'컨테이너의 프로세스는 어떤 사용자와 함께 시작 되었습니까? 포드는 POD 및 컨테이너 수준에서 정의 된 여러 컨테이너 및 보안 컨텍스트로 생성됩니다.

    ```yaml
    # multi-pod.yaml

    apiVersion: v1
    kind: Pod                                                                       .....................
    metadata:
      name: multi-pod
    spec:
      securityContext:
        runAsUser: 1001 # securityContext가 없는 이미지에 할당
      containers:
      -  image: ubuntu
         name: web
         command: ["sleep", "5000"]
         securityContext:
          runAsUser: 1002

      -  image: ubuntu
         name: sidecar
         command: ["sleep", "5000"]

    # 1001이 1002로 대체됨
    # 답은 1002
    ```

- '사이드카'컨테이너의 프로세스는 어떤 사용자와 함께 시작 되었습니까? 포드는 POD 및 컨테이너 수준에서 정의 된 여러 컨테이너 및 보안 컨텍스트로 생성됩니다.

    ```yaml
      -  image: ubuntu
         name: sidecar
         command: ["sleep", "5000"]

    # 얘는 따로 설정이 없으니 1001
    ```

- 포드 'ubuntu-sleeper'에서 아래 명령을 실행하여 날짜를 설정하십시오. POD에 날짜를 설정할 수 있습니까?

    date -s '19 APR 2012 11:14:00 '

    ```yaml
    kubectl exec -it ubuntu-sleeper -- bash
    date -s '19 APR 2012 11:14:00'

    # 설정 불가
    ```

- POD 'ubuntu-sleeper'를 업데이트하여 루트 사용자로 그리고 'SYS_TIME'기능으로 실행하십시오. 
참고 : 필요한 사항 만 변경하십시오. 포드 이름을 수정하지 마십시오.
    - Pod Name: ubuntu-sleeper
    - Image Name: ubuntu
    - SecurityContext: Capability SYS_TIME

    ```yaml
    # ubuntu-sleeper.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: ubuntu-sleeper
    spec:
      securityContext:
        runAsUser: 1001
      containers:
      - image: ubuntu
        name: ubuntu
        command: ["sleep", "4800"]
        securityContext:
          runAsUser: 1010
          capabilities:
            add: ["SYS_TIME"]
    ```

    ```bash
    kubectl delete pods ubuntu-sleeper
    kubectl create -f ubuntu-sleeper.yaml
    ```

    [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

- 이제 포드에서 아래 명령을 실행하여 날짜를 설정하십시오. 보안 기능이 올바르게 추가되면 작동합니다. 그것이 확실하지 않으면 사용자를 다시 루트로 변경하십시오.

    date -s '19 APR 2012 11:14:00 '

    ```bash
    kubectl exec -it ubuntu-sleeper -- bash
    date -s '19 APR 2012 11:14:00'
    ```