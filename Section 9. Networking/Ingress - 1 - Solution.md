# Ingress - 1 - Solution

# KEYWORD:
kubectl get ns /
kubectl get ingress

- Ingress 컨트롤러, 리소스 및 응용 프로그램을 배포했습니다. 설정을 탐색하십시오.

    ```bash
    kubectl get all --all-namespaces
    ```

- Ingress Controller는 어떤 네임 스페이스에 배포됩니까?

    ```bash
    kubectl get all --all-namespaces | grep -i controller

    # ingress-space
    ```

- Ingress Controller Deployment의 이름은 무엇입니까?

    ```bash
    kubectl get all --all-namespaces | grep -i controller | grep -i deploy

    # nginx-ingress-controller
    ```

- 응용 프로그램은 어떤 네임 스페이스에 배포됩니까?

    ```bash
    kubectl get all --all-namespaces | grep -i webapp

    # app-space
    ```

- app-space 네임 스페이스에 몇 개의 응용 프로그램이 배포되어 있습니까? 이 네임 스페이스의 deployment를 계산합니다.

    ```bash
    kubectl get ns
    kubectl get deploy -n app-space --no-headers | wc -l
    ```

- Ingress 리소스는 어떤 네임 스페이스에 배포됩니까?

    ```bash
    kubectl get ingress --all-namespaces

    # app-space
    ```

- Ingress Resource의 이름은?

    ```bash
    kubectl get ingress --all-namespaces

    # ingress-wear-watch
    ```

- Ingress Resource의 호스트는?

    ```bash
    kubectl get ingress --all-namespaces

    # *
    ```

- Ingress의 /wear 경로는 어떤 백엔드로 구성되어 있습니까?

    ```bash
    kubectl describe ingress ingress-wear-watch -n app-space

    # wear-service
    ```

- Ingress에서 비디오 스트리밍 응용 프로그램을 사용할 수있는 경로는 무엇입니까?

    ```bash
    kubectl describe ingress ingress-wear-watch -n app-space | grep -i video
    ```

- 요구 사항이 구성된 경로와 일치하지 않으면 요청이 어떤 서비스로 전달됩니까?

    ```bash
    kubectl describe ingress ingress-wear-watch -n app-space | grep -i default

    # Default backend:  default-http-backend:80 (<none>)
    ```

- 응용 프로그램을 사용할 수있는 URL을 변경해야합니다. /stream에서 비디오 응용 프로그램을 사용할 수있게하십시오.
    - Ingress: ingress-wear-watch
    - Path: /stream
    - Backend Service: video-service
    - Backend Service Port: 8080

    ```bash
    kubectl edit ingress ingress-wear-watch -n app-space

    # /watch -> /stream으로만 바꾸면 됨
    ```

    ```yaml
    # stream-ingress.yaml

    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: ingress-wear-watch
      namespace: app-space
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
        # redirection
    spec:
      rules:
      - http:
          paths:
          - backend:
              serviceName: wear-service
              servicePort: 8080
            path: /wear
          - backend:
              serviceName: video-service
              servicePort: 8080
            path: /stream
    ```

    [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- 음식 배달 응용 프로그램을 고객이 사용할 수 있도록 새로운 진입 경로를 추가해야합니다. 경로는 /eat
    - Ingress: ingress-wear-watch
    - Path: /eat
    - Backend Service: food-service
    - Backend Service Port: 8080\

    ```bash
    kubectl get ingress ingress-wear-watch -n app-space -o yaml > ingress-eat.yaml
    vim ingress-eat.yaml
    ```

    ```yaml
    # eat-ingress.yaml

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: ingress-wear-watch  
      namespace: app-space
      annotations:    
        nginx.ingress.kubernetes.io/rewrite-target: /    
        nginx.ingress.kubernetes.io/ssl-redirect: "false"
    spec:  
      rules:  
      - http:
          paths:
          - backend:
              serviceName: wear-service
              servicePort: 8080
            path: /wear
          - backend:
              serviceName: video-service
              servicePort: 8080
            path: /stream
          - backend:
              serviceName: food-service
              servicePort: 8080
            path: /eat
    ```

    [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- 새로운 결제 서비스가 도입되었습니다. 중요하기 때문에 새 응용 프로그램은 자체 네임 스페이스에 배포됩니다.

    ```bash
    kubectl get all --all-namespaces | grep -i pay
    kubectl get deploy --all-namespaces

    # critical-space
    ```

- 새로운 어플리케이션의 deployment 이름은?

    ```bash
    kubectl get deploy -n critical-space

    # webapp-pay
    ```

- 새 응용 프로그램을 /pay에 제공해야합니다.
    - Ingress Created
    - Path: /pay
    - Configure correct backend service
    - Configure correct backend port

    ```bash
    kubectl get svc -n critical-space
    # 8282/TCP
    ```

    ```yaml
    # vim ingress-pay.yaml

    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: ingress-pay
      namespace: critical-space
    # annotaions가 무조건 있어야 함.
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
        # host: rewrite.bar.com 이거 넣으면 안 됨
          paths:
          - path: /pay
            backend:
              serviceName: pay-service
              servicePort: 8282
    ```

    [Rewrite - NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/examples/rewrite/)

    [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- ingress-pay 확인 URL/pay

    ```bash
    kubectl delete ingress ingress-wear-watch -n app-space

    # 같은 80 포트라 삭제해줘야 함.
    ```