# KubeConfig - Solution

# KEYWORD:
cd $HOME/.kube /
kubectl config

- 현재 환경에서 기본 kubeconfig 파일은 어디에 있습니까? HOME 환경 변수를보고 현재 홈 디렉토리를 찾으십시오.

    ```bash
    ls -al
    cd $HOME/.kube # home directory is root
    pwd
    # /root/.kube/config
    ```

- 기본 kubeconfig 파일에 몇 개의 클러스터가 정의되어 있습니까?

    ```bash
    kubectl config view

    # cluster 개수 세기
    # 1개
    ```

- 기본 kubeconfig 파일에 몇 명의 사용자가 정의되어 있습니까?

    ```bash
    kubectl config view

    # users 보기
    # 1명
    ```

- 기본 kubeconfig 파일에 몇 개의 컨텍스트가 정의되어 있습니까?

    ```bash
    kubectl config view

    # context 보기 
    # 1개
    ```

- 현재 컨텍스트에서 사용자는 무엇입니까?

    ```bash
    kubectl config get-contexts 
    # kubernetes-admin@kubernetes

    kubectl config view | grep -i user
    # kubernetes-admin
    ```

- 기본 kubeconfig 파일에 구성된 클러스터 이름은 무엇입니까?

    ```bash
    kubectl config get-contexts
    # CLUSTER
    # kubernetes

    kubectl config view
    # kubernetes
    ```

- 'my-kube-config'라는 새 kubeconfig 파일이 작성됩니다. /root 디렉토리에 있습니다. kubectl 파일에 몇 개의 클러스터가 정의되어 있습니까?

    ```bash
    kubectl config get-contexts --kubeconfig=/root/my-kube-config
    # 4개

    # 또는
    kubectl config view --kubeconfig=/root/my-kube-config
    ```

    ```yaml
    # my-kube-config

    apiVersion: v1
    kind: Config

    clusters:
    - name: production
      cluster:
        certificate-authority: /etc/kubernetes/pki/ca.crt
        server: https://172.17.0.61:6443

    - name: development
      cluster:
        certificate-authority: /etc/kubernetes/pki/ca.crt
        server: https://172.17.0.61:6443

    - name: kubernetes-on-aws
      cluster:
        certificate-authority: /etc/kubernetes/pki/ca.crt
        server: https://172.17.0.61:6443

    - name: test-cluster-1
      cluster:
        certificate-authority: /etc/kubernetes/pki/ca.crt
        server: https://172.17.0.61:6443

    contexts:
    - name: test-user@development
      context:
        cluster: development
        user: test-user

    - name: aws-user@kubernetes-on-aws
      context:
        cluster: kubernetes-on-aws
        user: aws-user

    - name: test-user@production
      context:
        cluster: production
        user: test-user

    - name: research
      context:
        cluster: test-cluster-1
        user: dev-user

    users:
    - name: test-user
      user:
        client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
        client-key: /etc/kubernetes/pki/users/test-user/test-user.key
    - name: dev-user
      user:
        client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
        client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
    - name: aws-user
      user:
        client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
        client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

    current-context: test-user@development
    preferences: {}

    # 4개
    ```

- 'my-kube-config'파일에 몇 개의 컨텍스트가 구성되어 있습니까?

    ```bash
    kubectl config get-contexts --kubeconfig=/root/my-kube-config

    kubectl config view --kubeconfig=/root/my-kube-config

    # 4개
    # my-kube-config 파일은 위에 문제 참고
    ```

- research 컨텍스트에서 어떤 사용자가 구성됩니까?

    ```bash
    kubectl config get-contexts --kubeconfig=/root/my-kube-config
    # NAME        CLUSTER               AUTHINFO
    # research    test-cluster-1        dev-user

    kubectl config view --kubeconfig=/root/my-kube-config
    ```

    ```yaml
    - name: research
      context:
        cluster: test-cluster-1
        user: dev-user
    ```

- 'aws-user'에 대해 구성된 클라이언트 인증서 파일의 이름은 무엇입니까?

    ```bash
    kubectl config view --kubeconfig=/root/my-kube-config
    ```

    ```yaml
    - name: aws-user
      user:
        client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
        client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
    ```

- 'my-kube-config'파일에서 현재 컨텍스트는 무엇으로 설정되어 있습니까?

    ```bash
    kubectl config get-contexts
    # CURRENT에 * 있는 거 NAME

    # 또는
    kubectl config view --kubeconfig=/root/my-kube-config
    ```

    ```yaml
    # my-kube-config 맨 아래

    current-context: test-user@development
    ```

- `dev-user` 사용하여  `test-cluster-1`에 액세스 하고 싶습니다. 내가 할 수 있도록 현재 컨텍스트를 올바른 것으로 설정하십시오. 올바른 컨텍스트가 식별되면 'kubectl config use-context'명령을 사용하십시오.

    ```bash
    kubectl config view --kubeconfig=/root/my-kube-config

    # 정보 확인
    # - name: research
    #   context:
    #     cluster: test-cluster-1
    #     user: dev-user

    kubectl config get-contexts --kubeconfig=/root/my-kube-config

    kubectl config use-context \
        --kubeconfig=/root/my-kube-config \
        research # context name
    ```

- 현재 환경에서 기본 kubeconfig 파일을 my-kube-config로 교체

    ```bash
    cp my-kube-config $HOME/.kube
    cd $HOME/.kube
    mv config ..
    mv my-kube-config config
    ```

- 현재 컨텍스트가 리서치로 설정되어 클러스터에 액세스하려고합니다. 그러나 무언가 잘못된 것 같습니다. 문제를 식별하고 해결하십시오. `kubectl get pods`명령을 실행하고 오류를 찾으십시오. 모든 사용자 인증서는`/etc/kubernetes/pki/users`

    ```bash
    # 방법 1
    kubectl get pods
    # error: unable to read client-cert /etc/kubernetes/pki/users/dev-user/developer-user.crt 
    # for dev-user due to open /etc/kubernetes/pki/users/dev-user/developer-user.crt: no such file or directory

    cd /etc/kubernetes/pki/users/dev-user/
    ls
    # dev-user.crt  dev-user.csr  dev-user.key
    mv dev-user.crt developer-user.crt
    ```

    ```bash
    # 방법 2

    kubectl get pods
    cd $HOME/.kube
    vim config
    ```

    ```yaml
    - name: dev-user
      user:
        #client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
        client-certificate: /etc/kubernetes/pki/users/dev-user/dev-user.crt
        client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
    ```