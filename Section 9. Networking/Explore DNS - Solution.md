# Explore DNS - Solution

# KEYWORD:
kubectl get svc (svc name) /
kubectl get ns /
"svc" /
kubectl configmap (domain)

- 이 클러스터에서 구현 된 DNS 솔루션을 식별하십시오.

    ```bash
    kubectl get pods --all-namespaces | grep -i dns
    ```

- DNS 서버의 포드는 몇 개입니까?

    ```bash
    kubectl get pods -n kube-system | grep -i dns | wc -l
    ```

- CoreDNS에 액세스하기 위해 생성 된 서비스 이름은 무엇입니까?

    ```bash
    kubectl get svc -n kube-system | grep -i dns 
    ```

- 서비스를 해결하기 위해 POD에서 구성해야하는 CoreDNS 서버의 IP는 무엇입니까?

    ```bash
    kubectl get svc -n kube-system | grep -i dns

    # 10.96.0.10
    ```

- CoreDNS 서비스 구성 파일은 어디에 있습니까?

    ```bash
    kubectl get all --all-namespaces | grep -i coredns
    kubectl describe deploy coredns -n kube-system | grep -i -A3 args

    # 아래 두 개도 가능
    kubectl describe rs coredns-5644d7b6d9 -n kube-system | grep -i file
    kubectl describe pods coredns-5644d7b6d9-8x8w8 -n kube-system | grep -i file

    #Args:
    #      -conf
    #      /etc/coredns/Corefile
    ```

- Corefile은 어떻게 CoreDNS POD로 전달됩니까?

    ```bash
    kubectl describe pods coredns-5644d7b6d9-8x8w8 -n kube-system | grep -i type

    # configured as a ConfigMap object
    ```

- Corefile 용으로 생성 된 ConfigMap 객체의 이름은 무엇입니까?

    ```bash
    kubectl describe pods coredns-5644d7b6d9-8x8w8 -n kube-system | grep -i -A3 type

    # coredns

    # 또는
    kubectl get cm
    ```

![Untitled 1](https://user-images.githubusercontent.com/63388678/104575746-b9609900-569a-11eb-963c-d9e0978315c1.png)

- 이 kubernetes 클러스터에 대해 구성된 root domain/zone은 무엇입니까?

    ```bash
    kubectl get configmap --all-namespaces
    kubectl describe configmap coredns -n kube-system | grep -i kube

    # cluster.local
    ```

- 응용 프로그램 hr에서 web-service에 액세스하는 데 어떤 이름을 사용할 수 있는가? pod test에서 curl을 통해 확인 가능

    ```bash
    # web-service 확인
    kubectl get svc -o wide --all-namespaces

    # pod ip 확인
    # hr 10.88.0.4
    kubectl get pods -o wide

    # endpoint ip 확인
    # web-service    10.88.0.4:80
    kubectl get ep

    # 확인
    kubectl exec -it test -- sh
    curl http://web-service

    # web-service
    ```

- 테스트 포드에서 HR 서비스에 액세스하는 데 사용할 수없는 이름은 무엇입니까?

    ```bash
    [svc name] + [ns] + "svc" + [domain]

    # [svc name] 알기
    # web-service
    kubectl get svc --all-namespaces

    # [ns] 알기
    # default
    kubectl get ns

    # [domain] 알기
    # cluster.local
    kubectl get configmap --all-namespaces
    kubectl describe configmap coredns -n kube-system | grep -i kube

    kubectl exec -it test -- sh
    curl http://web-service.default.pod # 불가능

    # 가능, 
    # web-service // 네임스페이스가 default일 때만 가능
    # web-service.default
    # web-service.default.svc
    # web-service.default.svc.cluster.local
    ```

- 다음 중 테스트 응용 프로그램에서 payroll service 에 액세스하는 데 사용할 수있는 이름은 무엇입니까?

    ```bash
    [svc name] + [ns] + "svc" + [domain]

    # [svc name]
    # web-service
    kubectl get pods web -n payroll -o wide
    # 10.32.0.2
    kubectl get ep -n payroll
    # 10.32.0.2:80
    kubectl get svc -n payroll
    kubectl describe svc -n payroll
    # Endpoints:         10.32.0.2:80

    # [ns]
    # payroll
    kubectl get ns

    # [doamin]
    # cluster.local
    kubectl get configmap --all-namespaces
    kubectl describe configmap coredns -n kube-system | grep -i kube

    kubectl exec -it test -- sh
    curl web-service.payroll # 가능

    # 가능
    # web-service.payroll
    # web-service.payroll.svc
    # web-service.payroll.svc.cluster.local
    ```

- 다음 중 테스트 응용 프로그램에서 급여 서비스에 액세스하는 데 사용할 수없는 이름은 무엇입니까?

    ```bash
    [svc name] + [ns] + "svc" + [domain]

    # [svc name]
    # web-service
    kubectl get pods web -n payroll -o wide
    # 10.32.0.2
    kubectl get ep -n payroll
    # 10.32.0.2:80
    kubectl get svc -n payroll
    kubectl describe svc -n payroll
    # Endpoints:         10.32.0.2:80

    # [ns]
    # payroll
    kubectl get ns

    # [doamin]
    # cluster.local
    kubectl get configmap --all-namespaces
    kubectl describe configmap coredns -n kube-system | grep -i kube

    curl http://web-service.payroll.svc.cluster # 불가능

    # 가능
    # web-service.payroll
    # web-service.payroll.svc
    # web-service.payroll.svc.cluster.local
    ```

- 방금 `webapp`데이터베이스 `mysql`서버에 액세스 하는 웹 서버를 배포했습니다 . 그러나 웹 서버가 데이터베이스 서버에 연결하지 못했습니다. 문제를 해결하고 문제를 해결하십시오.

    다른 네임 스페이스에있을 수 있습니다. 먼저 응용 프로그램을 찾으십시오. `Web Server`터미널 상단의 탭 을 클릭하면 웹 서버 인터페이스를 볼 수 있습니다 .

    - Web Server: webapp
    - Uses the right DB_Host name

    ```bash
    # webapp-deploy.yaml

    [svc name] + [ns] + "svc" + [domain]

    # [svc name]
    # mysql
    kubectl get svc --all-namespaces
    kubectl describe svc mysql --all-namespaces

    # [ns]
    # payroll
    kubectl get ns

    # [doamin]
    # cluster.local
    kubectl get configmap --all-namespaces
    kubectl describe configmap coredns -n kube-system | grep -i kube

    # 가능
    # mysql.payroll
    # mysql.payroll.svc
    # mysql.payroll.svc.cluster.local

    kubectl get deploy
    kubectl get deploy webapp -o yaml > webapp-deploy.yaml
    vim webapp-deploy.yaml
    ```

    ```yaml
    # webapp-deploy.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:  
      labels:
        name: webapp
      name: webapp
      namespace: default
    spec:
      replicas: 1
      selector:
        matchLabels:
          name: webapp
      strategy:
        rollingUpdate:
          maxSurge: 25%
          maxUnavailable: 25%
        type: RollingUpdate
      template:
        metadata:
          creationTimestamp: null
          labels:
            name: webapp
        spec:
          containers:
          - env:
            - name: DB_Host
              # value: mysql->mysql.payroll로 수정함
              # 아래 다 가능
              value: mysql.payroll
              value: mysql.payroll.svc 
              value: mysql.payroll.svc.cluster.local
            - name: DB_User
              value: root
            - name: DB_Password
              value: paswrd
            image: mmumshad/simple-webapp-mysql
            name: simple-webapp-mysql
            ports:
            - containerPort: 8080
              protocol: TCP
    ```

    ```bash
    kubectl delete -f webapp-deploy.yaml
    kubectl create -f webapp-deploy.yaml
    ```

- pod hr에서 nslookup 실행

    ```bash
    kubectl exec -it hr nslookup mysql.payroll > /root/nslookup.out
    ```