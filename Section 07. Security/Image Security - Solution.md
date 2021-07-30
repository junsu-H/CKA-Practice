# Image Security - Solution

# KEYWORD:
kubectl create secret- h /
imagePullSecrets

- 클러스터에서 실행중인 응용 프로그램이 있습니다. 먼저 살펴 보겠습니다. 응용 프로그램에서 어떤 이미지를 사용하고 있습니까?

    ```bash
    kubectl get deploy
    kubectl describe deploy web | grep -i image

    # Image:          nginx:alpine
    ```

- 내부 개인 레지스트리에서 수정 된 버전의 응용 프로그램을 사용하기로 결정했습니다. 새 이미지를 사용하도록 배포 이미지를 업데이트하십시오.`myprivateregistry.com:5000` 레지스트리는에 있습니다 `myprivateregistry.com:5000`. 지금은 자격 증명에 대해 걱정하지 마십시오. 다음 단계에서 구성 할 것입니다.

    ```bash
    kubectl edit deploy web

    # 또는
    kubectl get deploy web -o yaml > nginx-deploy.yaml

    # Image:        myprivateregistry.com:5000/nginx:alpine
    ```

- 새 이미지로 생성 된 새 POD가 성공적으로 실행되고 있습니까?

    ```bash
    kubectl get pods

    # ErrImagePull
    ```

- 레지스트리에 액세스하는 데 필요한 자격 증명으로 비밀 객체를 만듭니다.

    Name:`private-reg-cred`

    Username:`dock_user`

    Password:`dock_password`

    Server:`myprivateregistry.com:5000`

    Email:`dock_user@myprivateregistry.com`

    - Secret: private-reg-cred
    - Secret Type: docker-registry
    - Secret Data

    ```bash
    kubectl create secret -h
    kubectl create secret docker-registry -h

    kubectl create secret docker-registry \
        [NAME] \
        --docker-username=userName \
        --docker-password=password \ 
        --docker-email=email \
        [--docker-server=string] \
        [--from-literal=key1=value1] \ 
        [--dry-run] \
        [options]

    kubectl create secret docker-registry \
        private-reg-cred \
        --docker-username=dock_user \
        --docker-password=dock_password \
        --docker-email=dock_user@myprivateregistry.com \
        --docker-server=myprivateregistry.com:5000 \
        --dry-run

    kubectl create secret docker-registry \
        private-reg-cred \
        --docker-username=dock_user \
        --docker-password=dock_password \
        --docker-email=dock_user@myprivateregistry.com \
        --docker-server=myprivateregistry.com:5000 
    ```

    [이미지](https://kubernetes.io/ko/docs/concepts/containers/images/)

- 새 비밀의 신임 정보를 사용하여 개인 레지스트리에서 이미지를 가져 오도록 배치를 구성하십시오.

    ```bash
    kubectl edit deploy web

    # 또는 

    kubectl get deploy web -o yaml > inner-nginx-deploy.yaml

    #      imagePullSecrets:
    #      - name: private-reg-cred
    ```

    ```yaml
    # inner-nginx-deploy.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      generation: 3
      labels:
        run: web
      name: web
      namespace: default
    spec:
      replicas: 2
      revisionHistoryLimit: 10
      selector:
        matchLabels:
          run: web
      strategy:
        rollingUpdate:
          maxSurge: 25%
          maxUnavailable: 25%
        type: RollingUpdate
      template:
        metadata:
          creationTimestamp: null
          labels:
            run: web
        spec:
          containers:
          - image: myprivateregistry.com:5000/nginx:alpine
            imagePullPolicy: IfNotPresent
            name: web
          imagePullSecrets:
          - name: private-reg-cred
    ```

    [이미지](https://kubernetes.io/ko/docs/concepts/containers/images/)

- POD 상태를 확인하십시오. 그들이 실행될 때까지 기다리십시오. 개인 레지스트리에서 이미지를 가져 오도록 배치를 성공적으로 구성했습니다.

    ```yaml
    watch "kubectl get pods -o wide"
    ```