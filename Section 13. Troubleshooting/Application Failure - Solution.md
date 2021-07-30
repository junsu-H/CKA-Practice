# Application Failure - Solution

# KEYWORD:
kubectl get pods /
kubectl get svc /
kubectl get deploy

![Untitled 1](https://user-images.githubusercontent.com/63388678/104706099-09546400-575e-11eb-8b6d-cbe198b33e97.png)

- 문제 해결 테스트 1 : `alpha` 네임 스페이스 에간단한 2 계층 응용 프로그램이 배포되었습니다 . 성공하면 녹색 웹 페이지가 표시되어야합니다. 터미널 상단의 앱 탭을 클릭하여 애플리케이션을 봅니다. 현재 실패했습니다. 문제를 해결하고 문제를 해결하십시오.

    ```bash
    # service name error

    kubectl get all -n alpha

    kubectl get svc -n alpha
    kubectl get svc mysql -n alpha -o yaml > mysql-service-alpha.yaml
    kubectl delete -f mysql-service-alpha.yaml

    # 또는
    kubectl edit deploy webapp-mysql -n alpha
    ```

    ```yaml
    # mysql-service-alpha.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: mysql-service
      namespace: alpha
    spec:
      clusterIP: 10.102.149.233
      ports:
      - port: 3306 # internal service port
        protocol: TCP
        targetPort: 3306 # internal Pod port    
      # nodePort: 30008 # external service port
      selector:
        name: mysql
      sessionAffinity: None
      type: ClusterIP
    ```

    ```yaml
    kubectl create -f mysql-service-alpha.yaml
    ```

- 문제 해결 테스트 2 :`beta` 네임 스페이스 에 동일한 2 계층 응용 프로그램이 배포되었습니다 . 성공하면 녹색 웹 페이지가 표시되어야합니다. 터미널 상단의 앱 탭을 클릭하여 애플리케이션을 봅니다. 현재 실패했습니다. 문제를 해결하고 문제를 해결하십시오. 주어진 아키텍처를 고수하십시오. 아래 아키텍처 다이어그램에 제공된 것과 동일한 이름과 포트 번호를 사용하십시오. 필요에 따라 개체를 자유롭게 편집, 삭제 또는 다시 작성하십시오.

    ```yaml
    # endpoint error

    kubectl get all -n beta
    kubectl edit deploy webapp-mysql -n beta

    kubectl edit svc mysql-service -n beta
    kubectl edit svc web-service -n beta

    # 다음과 같이 수정
    port: 8080 # 자기 자신 SERVICE 포트
    targetPort: 3306 # 뒷단(end-point)에 오는 POD 포트

    ```

- 문제 해결 테스트 3 :`gamma` 네임 스페이스 에 동일한 2 계층 응용 프로그램이 배포되었습니다 . 성공하면 녹색 웹 페이지가 표시되어야합니다. 터미널 상단의 앱 탭을 클릭하여 애플리케이션을 봅니다. 현재 실패했습니다. 문제를 해결하고 문제를 해결하십시오. 주어진 아키텍처를 고수하십시오. 아래 아키텍처 다이어그램에 제공된 것과 동일한 이름과 포트 번호를 사용하십시오. 필요에 따라 개체를 자유롭게 편집, 삭제 또는 다시 작성하십시오.

    ```bash
    # labels error

    kubectl get all -n gamma
    kubectl get ep -n gamma

    kubectl describe pod mysql -n gamma
    # labels:
    #   name: mysql

    kubectl get svc mysql-service -n gamma -o yaml > mysql-service-gamma.yaml
    kubectl delete -f mysql-service-gamma.yaml
    ```

    ```yaml
    # mysql-service-gamma.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: mysql-service
      namespace: gamma
    spec:
      clusterIP: 10.98.190.78
      ports:
      - port: 3306
        protocol: TCP
        targetPort: 3306
      selector:
        # name: sql0000001에서 아래와 같이 수정, pod labels과 동일한 라벨링
        name: mysql
      sessionAffinity: None
      type: ClusterIP
    ```

    ```bash
    kubectl create -f mysql-service-gamma.yaml
    ```

![Untitled 2](https://user-images.githubusercontent.com/63388678/104706102-0a859100-575e-11eb-9607-27a65f371e27.png)

- 문제 해결 테스트 4 :`delta` 네임 스페이스 에 동일한 2 계층 응용 프로그램이 배포됩니다 . 성공하면 녹색 웹 페이지가 표시되어야합니다. 터미널 상단의 앱 탭을 클릭하여 애플리케이션을 봅니다. 현재 실패했습니다. 문제를 해결하고 문제를 해결하십시오. 주어진 아키텍처를 고수하십시오. 아래 아키텍처 다이어그램에 제공된 것과 동일한 이름과 포트 번호를 사용하십시오. 필요에 따라 개체를 자유롭게 편집, 삭제 또는 다시 작성하십시오.

    ```bash
    # pod env error

    kubectl get all -n delta

    kubectl describe pods mysql -n delta # env.value 확인

    kubectl get deploy webapp-mysql -n delta
    kubectl edit deploy webapp-mysql -n delta

    # /DB_User: root로 수정하면 됨
    spec:
      containers:
      - env:
        - name: DB_User
          value: root
    ```

- 문제 해결 테스트 5 : `epsilon`네임 스페이스에 동일한 2 계층 응용 프로그램이 배포됩니다 . 성공하면 녹색 웹 페이지가 표시되어야합니다. 터미널 상단의 앱 탭을 클릭하여 애플리케이션을 봅니다. 현재 실패했습니다. 문제를 해결하고 문제를 해결하십시오. 주어진 아키텍처를 고수하십시오. 아래 아키텍처 다이어그램에 제공된 것과 동일한 이름과 포트 번호를 사용하십시오. 필요에 따라 개체를 자유롭게 편집, 삭제 또는 다시 작성하십시오.

    # 1. First

    ```bash
    # pod value error

    kubectl get pods -n epsilon
    kubectl describe pods mysql -n epsilon # MY_SQL_ROOT_PASSWORD:
    # Environment:
    #       MYSQL_ROOT_PASSWORD:  passwooooorrddd

    kubectl get pods mysql -o yaml -n epsilon > mysql-epsilon.yaml
    kubectl delete -f mysql-epsilon.yaml
    vim mysql-epsilon.yaml
    ```

    ```yaml
    # mysql-epsilon.yaml
    ...

    spec:
      containers:
      - env:
        - name: MYSQL_ROOT_PASSWORD
          # value: passwoooorddd 를 아래와 같이 수정
          value: paswrd
    ...
    ```

    ```bash
    kubectl create -f mysql-epsilon.yaml
    ```

    # 2. Second

    ```bash
    # deploy value error

    kubectl get all -n epsilon

    kubectl edit deploy webapp-mysql -n epsilon

    # /DB_User: root로 수정하면 됨
    spec:
      containers:
      - env:
        - name: DB_User
          value: root
    ```

- 문제 해결 테스트 6 :`zeta` 네임 스페이스 에 동일한 2 계층 응용 프로그램이 배포되었습니다 . 성공하면 녹색 웹 페이지가 표시되어야합니다. 터미널 상단의 앱 탭을 클릭하여 애플리케이션을 봅니다. 현재 실패했습니다. 문제를 해결하고 문제를 해결하십시오. 주어진 아키텍처를 고수하십시오. 아래 아키텍처 다이어그램에 제공된 것과 동일한 이름과 포트 번호를 사용하십시오. 필요에 따라 개체를 자유롭게 편집, 삭제 또는 다시 작성하십시오.

    # 1. First

    ```bash
    # pods env error

    kubectl describe pods mysql -n zeta # MY_SQL_ROOT_PASSWORD:
    # Environment:
    #       MYSQL_ROOT_PASSWORD:  passwooooorrddd

    kubectl get pods mysql -o yaml -n zeta > mysql-zeta.yaml
    kubectl delete -f mysql-zeta.yaml

    # pod는 kubectl edit으로 불가능
    ```

    ```yaml
    # mysql-zeta.yaml
    ...

    spec:
      containers:
      - env:
        - name: MYSQL_ROOT_PASSWORD
          # value: passwoooorddd 를 아래와 같이 수정
          value: paswrd
    ...
    ```

    ```bash
    kubectl create -f mysql-zeta.yaml
    ```

    # 2. Second

    ```bash
    # deploy value error

    kubectl get all -n zeta
    kubectl edit deploy webapp-mysql -n zeta

    # /DB_User: root로 수정
    spec:
      containers:
      - env:
        - name: DB_User
          value: root
    ```

    # 3. Third

    ```bash
    # service error

    kubectl get all -n zeta

    kubectl get svc web-service -n zeta
    kubectl get svc web-service -n zeta -o yaml > web-service-zeta.yaml
    kubectl delete -f web-service-zeta.yaml

    # 또는 
    kubectl edit svc web-service -n zeta
    ```

    ```yaml
    # web-service-zeta.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: web-service
      namespace: zeta
    spec:
      clusterIP: 10.106.33.38
      externalTrafficPolicy: Cluster
      ports:
      # - nodePort: 30088 에서 아래와 같이 수정
      - nodePort: 30081
        port: 8080
        protocol: TCP
        targetPort: 8080
      selector:
        name: webapp-mysql
      sessionAffinity: None
      type: NodePort
    ```

    ```bash
    kubectl create -f web-service-zeta.yaml
    ```