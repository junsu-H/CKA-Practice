# Ingress - 2 - Solution

# KEYWORD:
namespace /
configmap /
serviceaccount (role, rolebinding, clusterrole, clusterrolebinding) /
deployment /
service /
Ingress

```bash
1. CREATE NS
2. CREATE CM
3. CREATE SA
4. CREATE ROLE, ROLEBINDING, CLUSTERROLE, CLUSTERROLEBINDING
5. CREATE DEPLOYMENT
6. EXPOSE SERVICE
7. CREATE INGRESS
```

![Untitled 1](https://user-images.githubusercontent.com/63388678/104576084-1a886c80-569b-11eb-8ec5-9ae731c942e9.png)

```bash
# 이렇게 나오면 yaml 들여쓰기 문제 있는 거
error converting YAML to JSON: yaml: line 36: mapping values are not allowed in this context
```

- 이제 Ingress 컨트롤러를 배포하겠습니다. 먼저 'ingress-space'라는 네임 스페이스를 만듭니다. 모든 수신 관련 객체를 자체 네임 스페이스로 격리합니다. Name: ingress-space

    ```bash
    kubectl create namespace ingress-space
    ```

    [Share a Cluster with Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)

- NGINX Ingress 컨트롤러에는 ConfigMap 객체가 필요합니다. 수신 공간에 ConfigMap 오브젝트를 작성하십시오.  Name: nginx-configuration

    ```bash
    kubectl create configmap nginx-configuration -n ingress-space
    ```

    [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

- NGINX Ingress 컨트롤러에는 ServiceAccount가 필요합니다. 수신 공간에 ServiceAccount를 작성하십시오. Name: ingress-serviceaccount

    ```bash
    kubectl create sa ingress-serviceaccount -n ingress-space

    # role, rolebindings 관련된 거
    ```

    [Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

    ---

    ```yaml
    # ServiceAccount.yaml

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: ingress-serviceaccount
      namespace: ingress-space
    ```

    [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

- ServiceAccount에 대한 역할 및 역할 바인딩을 작성했습니다. 확인 해봐!

    ```bash
    kubectl get role -n ingress-space
    kubectl get rolebindings -n ingress-space
    kubectl get clusterrole -n ingress-space
    kubectl get clusterrolebinding -n ingress-space

    kubectl get role ingress-role -n ingress-space -o yaml
    kubectl get rolebindings ingress-role-binding -n ingress-space -o yaml
    kubectl get clusterrole ingress-clusterrole -n ingress-space -o yaml
    kubectl get clusterrolebinding ingress-clusterrole-binding -n ingress-space -o yaml
    ```

    ---

    ```yaml
    # ingress-role.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      name: ingress-role
      namespace: ingress-space
    rules:
    - apiGroups:
      - ""
      resources:
      - configmaps
      - pods
      - secrets
      - namespaces
      verbs:
      - get
    - apiGroups:
      - ""
      resourceNames:
      - ingress-controller-leader-nginx
      resources:
      - configmaps
      verbs:
      - get
      - update
    - apiGroups:
      - ""
      resources:
      - configmaps
      verbs:
      - create
    - apiGroups:
      - ""
      resources:
      - endpoints
      verbs:
      - get

    --- 
    # ingress-role-binding

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      name: ingress-role-binding
      namespace: ingress-space
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: ingress-role
    subjects:
    - kind: ServiceAccount
      name: ingress-serviceaccount
    ```

    ```yaml
    # ingress-clusterrole.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      name: ingress-clusterrole
      # namespace 없음
    rules:
    - apiGroups:
      - ""
      resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
      verbs:
      - list
      - watch
    - apiGroups:
      - ""
      resources:
      - nodes
      verbs:
      - get
    - apiGroups:
      - ""
      resources:
      - services
      verbs:
      - get
      - list
      - watch
    - apiGroups:
      - extensions
      resources:
      - ingresses
      verbs:
      - get
      - list
      - watch
    - apiGroups:
      - ""
      resources:
      - events
      verbs:
      - create
      - patch
    - apiGroups:
      - extensions
      resources:
      - ingresses/status
      verbs:
      - update

    ---
    # ingress-clusterrolebinding.yaml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      name: ingress-clusterrole-binding
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: ingress-clusterrole
    subjects:
    - kind: ServiceAccount
      name: ingress-serviceaccount
      namespace: ingress-space
    ```

- 이제 Ingress 컨트롤러를 배포하겠습니다. 주어진 파일을 사용하여 배치를 작성하십시오. 배포 구성은 /root/ingress-controller.yaml에 있습니다. 그것에 몇 가지 문제가 있습니다. 그것들을 고치십시오.
    - Deployed in the correct namespace.
    - Replicas: 1
    - Use the right image

    ```yaml
    # ingress-controller.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:  
      name: ingress-controller
      namespace: ingress-space
    spec:
      replicas: 1  
      selector:
        matchLabels:
          name: nginx-ingress
      template:
        metadata:
          labels:
            name: nginx-ingress # 얘랑 매칭
        spec:
          serviceAccountName: ingress-serviceaccount
          containers:
            - name: nginx-ingress-controller
              image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
              args:
                - /nginx-ingress-controller
                - --configmap=$(POD_NAMESPACE)/nginx-configuration
                - --default-backend-service=app-space/default-http-backend
              env:
                - name: POD_NAME
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.name
                - name: POD_NAMESPACE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.namespace
              ports:
                - name: http
                  containerPort: 80
                - name: https
                  containerPort: 443
    ```

    [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

- 이제 외부 사용자가 Ingress를 사용할 수 있도록 service를 작성하겠습니다.
    - Name: ingress
    - Type: NodePort
    - Port: 80
    - TargetPort: 80
    - NodePort: 30080
    - Use the right selector

    ```bash
    cat ingress-controller.yaml | grep -i selector -A5

    # selector:
    #     matchLabels:      name: nginx-ingress
    ```

    ```yaml
    # ingress-service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: ingress
      namespace: ingress-space
    spec:
      type: NodePort
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          nodePort: 30080
      selector:
        name: nginx-ingress # ingress-controller.yaml 얘 레이블이랑 매칭

    ```

    ```bash
    kubectl create -f ingress-service.yaml

    kubectl expose deploy ingress-controller --name=ingress \
        -n ingress-space \
        --port=80 \
        --target-port=80 \
        --type=NodePort \
        --selector=name=nginx-ingress \
        --dry-run

    kubectl edit svc ingress -n ingress-space
    > nodePort: 30080으로 수정
    ```

- Ingress 서비스의 /wear 및 /watch에서 응용 프로그램을 사용할 수 있도록 수신 리소스를 만듭니다. 앱 공간에서 수신을 만듭니다.
    - Ingress Created
    - Path: /wear
    - Path: /watch
    - Configure correct backend service for /wear
    - Configure correct backend service for /watch
    - Configure correct backend port for /wear service
    - Configure correct backend port for /watch service

    ```bash
    kubectl get svc -n app-space
    # service listen 8080

    kubectl get ep -n app-space
    # service 이름 확인, 8080 포트 확인
    ```

    ```yaml
    #  ingress-resource.yaml

    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: ingress-wear-watch
      namespace: app-space
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    #   nginx.ingress.kubernetes.io/ssl-redirect: "false"

    spec:
      rules:
      - http:
          paths:
          - path: /wear
            backend:
              # kubectl get svc -n app-space
              serviceName: wear-service
              servicePort: 8080
          - path: /watch
            backend:
              # kubectl get svc -n app-space
              serviceName: video-service
              servicePort: 8080
    ```

    ```bash
    kubectl create -f ingress-resource.yaml
    ```

    [Rewrite - NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/examples/rewrite/)