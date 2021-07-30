# Network Policy - Solution

# KEYWORD:
kubectl get netpol /
NetworkPolicy

- 환경에 몇 개의 네트워크 정책이 있습니까? 우리는 웹 응용 프로그램, 서비스 및 네트워크 정책을 거의 배포하지 않았습니다. 환경을 점검하십시오.

    ![Untitled 1](https://user-images.githubusercontent.com/63388678/104254058-59f76300-54b9-11eb-87aa-b97c6656b4e3.png)

    ```bash
    kubectl get netpol

    # 또는
    kubectl get networkpolicy

    # 1개
    ```

- 네트워크 정책의 이름은 무엇입니까?

    ![Untitled 2](https://user-images.githubusercontent.com/63388678/104254059-5b289000-54b9-11eb-8d17-8c7b7acc9711.png)

    ```bash
    kubectl get netpol

    # 또는
    kubectl get networkpolicy

    # payroll-policy
    ```

- 네트워크 정책이 적용되는 포드는 무엇입니까?

    ![Untitled 3](https://user-images.githubusercontent.com/63388678/104254060-5bc12680-54b9-11eb-9162-546529ef3b7d.png)

    ```bash
    kubectl describe netpol payroll-policy | grep -i pod

    # 또는
    kubectl describe networkpolicy payroll-policy | grep -i pod

    # PodSelector:     name=payroll
    ```

- 이 네트워크 정책은 어떤 유형의 트래픽을 처리하도록 구성되어 있습니까?

    ![Untitled 4](https://user-images.githubusercontent.com/63388678/104254061-5c59bd00-54b9-11eb-813f-764357502230.png)

    ```bash
    kubectl describe netpol payroll-policy | grep -i policy

    # Policy Types: Ingress
    ```

- 이 네트워크 정책에 구성된 규칙의 영향은 무엇입니까?

    ![Untitled 5](https://user-images.githubusercontent.com/63388678/104254062-5c59bd00-54b9-11eb-823a-f9b101d2df7b.png)

    ```bash
    kubectl describe netpol payroll-policy
    # Spec:
    #  PodSelector:     name=payroll
    #  Allowing ingress traffic:
    #    To Port: 8080/TCP
    #    From:
    #      PodSelector: name=internal
    #  Allowing egress traffic:
    #    <none> (Selected pods are isolated for egress connectivity)
    #  Policy Types: Ingress

    # 답
    # Traffic From Internal to Payroll POD is allowed
    ```

- 이 네트워크 정책에 구성된 규칙의 영향은 무엇입니까?

    ![Untitled 6](https://user-images.githubusercontent.com/63388678/104254253-c7a38f00-54b9-11eb-9d66-cb38a9d32b92.png)

    ```bash
    kubectl describe netpol payroll-policy
    # Spec:
    #  PodSelector:     name=payroll
    #  Allowing ingress traffic:
    #    To Port: 8080/TCP
    #    From:
    #      PodSelector: name=internal
    #  Allowing egress traffic:
    #    <none> (Selected pods are isolated for egress connectivity)
    #  Policy Types: Ingress

    # 답
    # Internal POD can access port 8080 on Payroll POD
    ```

- '내부'애플리케이션에서 'payroll-service'및 'db-service'로의 트래픽 만 허용하는 네트워크 정책을 작성하십시오. 오른쪽에 주어진 사양을 사용하십시오. UI에서 규칙을 테스트하기 위해 포드로 들어오는 트래픽을 활성화 할 수 있습니다.
    - Policy Name: internal-policy
    - Policy Types: Egress
    - Egress Allow: payroll
    - Payroll Port: 8080
    - Egress Allow: mysql
    - MYSQL Port: 3306

    ![Untitled 7](https://user-images.githubusercontent.com/63388678/104254254-c8d4bc00-54b9-11eb-9d93-07eec2b6f31d.png)

    ```bash
    # Labels 확인

    kubectl get pods

    kubectl get pods internal --show-labels
    # name=internal

    kubectl get pods mysql --show-labels
    # name=mysql

    kubectl get pods payroll --show-labels
    # name=payroll
    ```

    ```yaml
    # internal-policy.yaml

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:  
      name: internal-policy
      namespace: default
    spec:
      podSelector:
        matchLabels:
          name: internal
      policyTypes:
      - Egress
      - Ingress
      ingress:
      - {}
      egress:
      - to:
        - podSelector:
            matchLabels:
              name: mysql
        ports:
        - protocol: TCP
          port: 3306

      - to:
        - podSelector:        
            matchLabels:
              name: payroll
        ports:
        - protocol: TCP
          port: 8080
    ```

    [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)