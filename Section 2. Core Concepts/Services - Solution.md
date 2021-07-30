# Services - Solution

# KEYWORD:
kubectl get svc --show-labels /
kubectl get ep

- service 개수

    ```bash
    kubectl get svc
    ```

- 해당 svc에 targetPort는?

    ```bash
    kubectl get svc
    kubectl describe svc kubernetes | grep -i targetPort

    # target 6443
    ```

- 해당 svc에 Labels의 개수는?

    ```bash
    kubectl describe svc kubernetes | grep -i -A5 labels

    # 또는
    kubectl get svc --show-labels
    ```

- 해당 svc에 Endpoints의 개수는?

    ```bash
    kubectl describe svc | grep -i endpoints

    # 또는
    kubectl get ep
    ```

- Deployment의 개수는?

    ```bash
    kubectl get deploy
    ```

- Deployment의 image는?

    ```bash
    kubectl get deploy -o wide
    kubectl describe deployment
    ```

- service-definition-1.yaml을 작성하라

    Name: webapp-service; 

    Type: NodePort; 

    targetPort: 8080; 

    port: 8080; 

    nodePort: 30080; 

    selector: simple-webapp

    ```yaml
    # service-definition-1.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: webapp-service
    spec:
      selector:
        name: simple-webapp
      ports:
        - targetPort: 8080
          port: 8080
          nodePort: 30080
      type: NodePort
    ```

    ```bash
    kubectl create -f service-definition-1.yaml
    kubectl get deploy
    # 4개

    kubectl get pods -o wide
    # 10.32.0.2
    # 10.32.0.5
    # 10.32.0.4
    # 10.32.0.3

    kubectl get ep
    # webapp-service   10.32.0.2:8080,
    #                  10.32.0.3:8080,
    #                  10.32.0.4:8080 
    #                  + 1 more...

    # ep 4개 생성됨. (파드 개수만큼 생긴다.)
    # seletor를 생성하여 service를 만들면
    # ep는 pod ip:targetPort로 생성됨.
    # ep는 url에 접속하면 접속 가능한 것임.

    kubectl get ep webapp-service -o yaml > webapp-service-ep.yaml
    ```

    ```yaml
    # webapp-service-ep.yaml

    apiVersion: v1
    kind: Endpoints
    metadata:
      name: webapp-service
    subsets:
    - addresses:
      - ip: 10.32.0.4 # URL로 접근 가능한 곳
        nodeName: node01
        targetRef:
          kind: Pod
          name: simple-webapp-deployment-5d5b98455-ddmm8
          namespace: default
      - ip: 10.32.0.5 # URL로 접근 가능한 곳
        nodeName: node01
        targetRef:
          kind: Pod
          name: simple-webapp-deployment-5d5b98455-qmh2n
          namespace: default
      - ip: 10.32.0.6 # URL로 접근 가능한 곳
        nodeName: node01
        targetRef:
          kind: Pod
          name: simple-webapp-deployment-5d5b98455-7pwb8
          namespace: default
      - ip: 10.32.0.7 # URL로 접근 가능한 곳
        nodeName: node01
        targetRef:
          kind: Pod
          name: simple-webapp-deployment-5d5b98455-l7w4x
          namespace: default
      ports:
      - port: 8080
        protocol: TCP
    ```

    ```yaml
    # seletor없이 수동으로 service를 만들 수 있다.
    # 이렇게 만들면 url에 11.11.11.11:80으로 접속하면 pod랑 연동된다.

    # external-service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: external-service
    spec:
      ports:
      - port: 80
    ---

    # external-service-ep.yaml

    apiVersion: v1
    kind: Endpoints
    metadata:
      nmae: external-service
    subsets:
      - addresses:
        - ip: 11.11.11.11 # URL로 접근 가능한 곳
        - ip: 22.22.22.22 # URL로 접근 가능한 곳
        ports: 80
    ```

    ```bash
    # 구조는 다음과 같다.
    Pod --- SVC --- EP (EP는 URL 접근 가능)
    ```

![Untitled 1](https://user-images.githubusercontent.com/63388678/104253550-1cdea100-54b8-11eb-8aa7-12f29e10f30c.png)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort
  ports:
  - tagetPort: 80   # internal Pod port
    port: 80        # internal service port
    nodePort: 30008 # external service port
```