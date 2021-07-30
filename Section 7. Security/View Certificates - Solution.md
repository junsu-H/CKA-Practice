# View Certificates - Solution

# KEYWARD: 
/etc/kubernets/manifest  /
/etc/kubernets/pki /
openssl

- kube-apiserver에 사용된 인증서 파일을 식별

    ```bash
    # 일단 문제를 보고 뒤에 확장자 생각하기 .crt

    kubectl get all --all-namespaces
    kubectl get pods kube-apiserver-master -n kube-system
    kubectl describe pods kube-apiserver-master -n kube-system | grep -i crt

    # 또는
    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i apiserver

    # /etc/kubernetes/pki/apiserver.crt
    ```

- 서버 kube-apiserver에서 클라이언트로 인증 하는 데 사용되는 인증서 파일 ETCD

    ```bash
    # 일단 문제를 보고 뒤에 확장자 생각하기 .crt

    kubectl get all --all-namespaces
    kubectl get pods kube-apiserver-master -n kube-system
    kubectl describe pods kube-apiserver-master -n kube-system | grep -i crt | grep -i etcd

    # 또는
    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i etcd

    # /etc/kubernetes/pki/apiserver-etcd-client.crt
    ```

- 서버 인증 kubeapi-server에 사용되는 키 식별kubelet

    ```bash
    # 일단 문제를 보고 뒤에 확장자 생각하기 .key

    kubectl get all --all-namespaces
    kubectl get pods kube-apiserver-master -n kube-system
    kubectl describe pods kube-apiserver-master -n kube-system | grep -i kubelet | grep -i key

    # 또는
    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i kubelet

    # /etc/kubernetes/pki/apiserver-kubelet-client.key
    ```

- ETCD server를 호스팅하는 데 사용 된 ETCD 서버 인증서 식별

    ```bash
    # 일단 문제를 보고 뒤에 확장자 생각하기 .crt

    kubectl get all --all-namespaces | grep -i etcd
    kubectl get pods etcd-master -n kube-system
    kubectl describe pods etcd-master -n kube-system | grep -i server

    # 또는
    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    cat /etc/kubernetes/manifests/etcd.yaml | grep -i etcd

    # /etc/kubernetes/pki/etcd/server.crt
    ```

- ETCD 서버를 제공하는 데 사용 된 ETCD 서버 CA 루트 인증서 식별, ETCD는 자체 CA를 가질 수 있습니다. 따라서 이것은 kube-api 서버가 사용하는 것과 다른 CA 인증서 일 수 있습니다.

    ```bash
    # 다른 CA 인증서라는 것은 
    # vim /etc/kubernetes/manifests/kube-apiserver.yaml에 정보가 없다는 것.

    kubectl get all --all-namespaces | grep -i etcd
    kubectl get pods etcd-master -n kube-system
    kubectl describe pods etcd-master -n kube-system | grep -i ca

    # 또는
    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    cat /etc/kubernetes/manifests/etcd.yaml | grep -i etcd

    # --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    ```

- kube-apiserver 인증서에 구성된 CN (Common Name)은 무엇입니까?

    ```bash
    # OpenSSL 구문 : openssl x509 -in file-path.crt -text -noout

    kubectl get all --all-namespaces
    kubectl get pods kube-apiserver-master -n kube-system
    kubectl describe pods kube-apiserver-master -n kube-system | grep -i crt
    # /etc/kubernetes/pki/apiserver.crt

    # 또는
    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i apiserver
    # /etc/kubernetes/pki/apiserver.crt

    openssl x509 -h
    openssl x509 -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep CN

    # -noout 인증서 출력 안 함.
    # -text 텍스트로 출력
    # -in .crt 파일

    # Subject: CN=kube-apiserver
    ```

- kube-apiserver 인증서를 발행한 CA의 이름은 무엇입니까?

    ```bash
    openssl x509 -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep -i issuer

    # -noout 인증서 출력 안 함.
    # -text 텍스트로 출력
    # -in .crt 파일

    # Issuer: CN=kubernetes
    ```

    [PKI 인증서 및 요구 조건](https://kubernetes.io/ko/docs/setup/best-practices/certificates/)

    [Certificates](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)

- 다음 대체 이름 중 Kube API Server 인증서에 구성되지 않은 대체 이름은 무엇입니까?

    ```bash
    openssl x509 -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep -i -A1 alter

    # kube-master
    ```

- ETCD 서버 인증서에 구성된 CN (Common Name)은 무엇입니까?

    ```bash
    kubectl get all --all-namespaces | grep -i etcd
    kubectl get pods etcd-master -n kube-system
    kubectl describe pods etcd-master -n kube-system | grep -i server

    # 또는
    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    cat /etc/kubernetes/manifests/etcd.yaml | grep -i server

    openssl x509 -h
    openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text  | grep -i cn

    # Subject: CN=master
    ```

- 발행 날짜로부터 kube-apiserver 인증서는 얼마동안 유효합니까?

    파일 경로: /etc/kubernetes/pki/apiserver.crt

    ```bash
    openssl x509 -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep -i -A5 v
    alid

    # 1년
    ```

- 발급된 날짜부터 루트 CA 인증서는 얼마 동안 유효합니까?

    파일 경로: /etc/kubernetes/pki/ca.crt

    ```bash
    openssl x509 -noout -text -in /etc/kubernetes/pki/ca.crt | grep -i -A5 valid

    # 10년
    ```

- Kubectl이 갑자기 명령에 응답하지 않습니다. 누군가 `/etc/kubernetes/manifests/etcd.yaml` 파일을 수정했습니다.
문제를 조사하고 해결하라는 메시지가 표시됩니다. 문제를 해결하면 kubectl이 응답할 때까지 기다리십시오. ETCD 컨테이너의 로그를 확인하십시오.

    ```bash
    docker ps -a | grep -i etcd
    docker logs 0f2ca69fd6c2

    # etcdmain: open /etc/kubernetes/pki/etcd/server-certificate.crt: no such file or director

    ls /etc/kubernetes/pki/etcd/ | grep -i server
    # server.crt
    # server.key

    # server-certificate.crt는 존재하지 않는다.
    vim /etc/kubernetes/manifests/etcd.yaml

    # - --cert-file=/etc/kubernetes/pki/etcd/server-certificate.crt
    # 아래와 같이 수정

    - --cert-file=/etc/kubernetes/pki/etcd/server.crt

    watch "docker ps -a | grep -i etcd"

    # journalctl -u etcd.service -l

    # kubectl logs etcd-master
    ```

- kube-api 서버가 다시 중지되었습니다! kube-apiserver 로그를 검사하고 근본 원인을 식별하고 문제를 해결하십시오.`docker ps -a`kube-api 서버 컨테이너를 식별하는 명령을 실행하십시오 . `docker logs container-id`로그를 보려면 명령을 실행하십시오. kube-apiserver 수정

    ```bash
    docker ps -a | grep -i apiserver
    docker logs d0883139a4bf # 로그 봐도 문제가 뭔지 모름;;

    # W0417 grpc: addrConn.createTransport failed to connect to 
    # {https://127.0.0.1:2379 0  <nil>}. => etcd에 문제가 있다. (2379 포트는 etcd 포트)
    # Err :connection error: desc = 
    # "transport: authentication handshake failed:
    # x509: certificate signed  => etcd certificate에 문제가 있다.
    # by unknown authority". 
    # Reconnecting...
    ```

    ```yaml
    # vim /etc/kubernetes/manifests/kube-apiserver.yaml
    #   - --etcd-cafile=/etc/kubernetes/pki/ca.crt에서 etcd 추가
    # etcd는 etcd가 꼭 들어감

    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null  
      labels:
        component: kube-apiserver
        tier: control-plane
      name: kube-apiserver
      namespace: kube-system
    spec:
      containers:
      - command:
        - kube-apiserver
        - --advertise-address=172.17.0.16
        - --allow-privileged=true
        - --authorization-mode=Node,RBAC
        - --client-ca-file=/etc/kubernetes/pki/ca.crt
        - --enable-admission-plugins=NodeRestriction
        - --enable-bootstrap-token-auth=true
        - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
        - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
        - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
        - --etcd-servers=https://127.0.0.1:2379
        - --insecure-port=0
        - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
        - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
        - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
        - --requestheader-allowed-names=front-proxy-client
        - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
        - --requestheader-extra-headers-prefix=X-Remote-Extra-
        - --requestheader-group-headers=X-Remote-Group
        - --requestheader-username-headers=X-Remote-User
        - --secure-port=6443
        - --service-account-key-file=/etc/kubernetes/pki/sa.pub
        - --service-cluster-ip-range=10.96.0.0/12
        - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
        - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
        image: k8s.gcr.io/kube-apiserver:v1.16.0
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 8
          httpGet:
            host: 172.17.0.16
            path: /healthz
            port: 6443
            scheme: HTTPS
          initialDelaySeconds: 15
          timeoutSeconds: 15
        name: kube-apiserver
        resources:
          requests:
            cpu: 250m
        volumeMounts:
        - mountPath: /etc/ssl/certs
          name: ca-certs
          readOnly: true
        - mountPath: /etc/ca-certificates
          name: etc-ca-certificates
          readOnly: true
        - mountPath: /etc/pki
          name: etc-pki
          readOnly: true
        - mountPath: /etc/kubernetes/pki
          name: k8s-certs
          readOnly: true
        - mountPath: /usr/local/share/ca-certificates
          name: usr-local-share-ca-certificates
          readOnly: true
        - mountPath: /usr/share/ca-certificates
          name: usr-share-ca-certificates
          readOnly: true
      hostNetwork: true
      priorityClassName: system-cluster-critical
      volumes:
      - hostPath:
          path: /etc/ssl/certs
          type: DirectoryOrCreate
        name: ca-certs
      - hostPath:
          path: /etc/ca-certificates
          type: DirectoryOrCreate
        name: etc-ca-certificates
      - hostPath:
          path: /etc/pki
          type: DirectoryOrCreate
        name: etc-pki
      - hostPath:
          path: /etc/kubernetes/pki
          type: DirectoryOrCreate
        name: k8s-certs
      - hostPath:
          path: /usr/local/share/ca-certificates
          type: DirectoryOrCreate
        name: usr-local-share-ca-certificates
      - hostPath:
          path: /usr/share/ca-certificates
          type: DirectoryOrCreate
        name: usr-share-ca-certificates
    status: {}
    ```

- kube-api 서버가 ETCD 서버에 연결하려고하면 언젠가 응답하지 않습니다. 문제를 조사하고 수정하십시오. 인증서가 만료 된 경우 CA에서 새 인증서에 서명하고 kube-api 서버에서 사용하도록 구성하십시오.

    ```bash
    docker ps -a | grep -i api
    docker logs b317f572c10e
    # tls: bad certificate". Reconnecting... => .crt 파일이 이상하다.

    ps -aux | grep -i kubelet
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests
    cat kube-apiserver.yaml

    # - --client-ca-file=/etc/kubernetes/pki/ca.crt
    # - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    # - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    # - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    # - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt

    # .key는 oepnssl로 확인 불가
    # - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    # - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    # - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    ```

    ```bash
    # /etc/kubernetes/pki/ca.crt (정상)
    openssl x509 -in /etc/kubernetes/pki/ca.crt \
        -text | grep -i -A5 valid
    # Not Before: Apr 30 05:08:47 2020 GMT
    # Not After : Apr 28 05:08:47 2030 GMT

    # /etc/kubernetes/pki/etcd/ca.crt (정상)
    openssl x509 -in /etc/kubernetes/pki/etcd/ca.crt \
        -text | grep -i -A5 valid
    # Not Before: Apr 30 05:08:51 2020 GMT
    # Not After : Apr 28 05:08:51 2030 GMT

    # /etc/kubernetes/pki/apiserver-etcd-client.crt (비정상)
    openssl x509 -in /etc/kubernetes/pki/apiserver-etcd-client.crt \
        -text | grep -i -A5 valid
    # Not Before: Apr 30 05:38:16 2020 GMT
    # Not After : Apr 20 05:38:16 2020 GMT

    # /etc/kubernetes/pki/apiserver-kubelet-client.crt (정상)
    openssl x509 -in /etc/kubernetes/pki/apiserver-kubelet-client.crt \
        -text | grep -i -A5 valid
    # Not Before: Apr 30 05:08:47 2020 GMT
    # Not After : Apr 30 05:08:49 2021 GMT

    # /etc/kubernetes/pki/apiserver.crt (정상)
    openssl x509 -in /etc/kubernetes/pki/apiserver.crt \
        -text | grep -i -A5 valid
    # Not Before: Apr 30 05:08:47 2020 GMT
    # Not After : Apr 30 05:08:48 2021 GMT

    # /etc/kubernetes/pki/apiserver-etcd-client.crt 얘가 만료됨.
    openssl x509 -in /etc/kubernetes/pki/apiserver-etcd-client.crt \
        -text | grep -i -A5 valid
    # Not Before: Apr 30 05:38:16 2020 GMT
    # Not After : Apr 20 05:38:16 2020 GMT
    ```

    ```bash
    # 사용법

    openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out server.crt -days 10000 \
    -extensions v3_ext -extfile csr.conf

    find / -name "*.csr"
    ```

    ```bash
    openssl x509 -req -in \
        /etc/kubernetes/pki/apiserver-etcd-client.csr \
        -CA /etc/kubernetes/pki/etcd/ca.crt \
        -CAkey /etc/kubernetes/pki/etcd/ca.key \
        -CAcreateserial -out \
        /etc/kubernetes/pki/apiserver-etcd-client.crt # 갱신 필요한 정보 적기
    ```

    ```bash
    openssl x509 -in /etc/kubernetes/pki/apiserver-etcd-client.crt \
        -text | grep -i -A5 valid
    ```

    [Certificates](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)