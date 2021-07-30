# Control Plane Failure - Solution

# KEYWORD:
kubectl get all --all-namespaces /
kubectl logs [pod] /
ps -aux | grep -i config

- 클러스터가 다시 끊어졌습니다. 응용 프로그램 배포를 시도했지만 작동하지 않습니다. 문제를 해결하고 문제를 해결하십시오.

    ```bash
    # kube-system pod에 이상이 있으면 staticPath 확인

    kubectl get all --all-namespaces # kube-system pod 문제 확인

    # kubectl describe pods app-f54ccc97b-9bcs6

    # kubectl describe pods kube-scheduler-master -n kube-system

    ps -aux | grep -i config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cat /var/lib/kubelet/config.yaml | grep -i staticPodPath
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests
    vim kube-scheduler.yaml
    ```

    ```yaml
    # kube-scheduler.yaml

    spec:
      containers:
      - command:
        # - kube-schedulerrrr 아래와 같이 수정
        - kube-scheduler
    ```

    ```bash
    # 확인
    kubectl get pods -n kube-system
    ```

- 배포 app를 2 개의 포드로 확장합니다 .

    ```bash
    kubectl scale deploy app --replicas=2

    # 아래에서 replicas=2로 수정
    kubectl edit deploy app
    ```

- deployment가 2로 조정되었지만 POD 수는 증가하지 않는 것 같습니다. 문제를 조사하고 수정하십시오. 배포 및 복제 세트 관리를 담당하는 구성 요소를 검사하십시오.

    ```bash
    kubectl get all --all-namespaces
    # kube-controller-manager-master
    # 뒤에 master가 들어가면 static pod

    ps -aux | grep -i config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cat /var/lib/kubelet/config.yaml | grep -i staticPodPath
    # staticPodPath: /etc/kubernetes/manifests

    # 로그 확인
    kubectl describe pods kube-controller-manager-master -n kube-system
    kubectl logs kube-controller-manager-master -n kube-system
    # stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory

    cd /etc/kubernetes/manifests/
    vim kube-controller-manager.yaml
    ```

    ```yaml
    # vim kube-controller-manager.yaml
    # q!
    # ls /etc/kubernetes/
    # mountPath 확인

    # - mountPath: /etc/kubernetes/controller-manager-XXX.conf 아래와 같이 수정
    - mountPath: /etc/kubernetes/controller-manager.conf
          name: kubeconfig
          readOnly: true
    ```

    ```yaml
    # 확인
    kubectl get pods -n kube-system
    ```

- 다시 스케일링에 문제가 있습니다. 방금 배포를 3 개의 복제본으로 확장하려고했습니다. 그러나 일어나지 않습니다. 문제를 조사하고 해결하십시오

    ```bash
    kubectl get all --all-namespaces
    # kube-controller-manager-master
    # 뒤에 master가 들어가면 static pod

    ps -aux | grep -i config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cat /var/lib/kubelet/config.yaml | grep -i staticPodPath
    # staticPodPath: /etc/kubernetes/manifests

    # 로그 확인
    kubectl describe pods kube-controller-manager-master -n kube-system
    kubectl logs kube-controller-manager-master -n kube-system
    # unable to load client CA file: unable to load client CA file: open /etc/kubernetes/pki/ca.crt: no such file or directory

    cd /etc/kubernetes/manifests/
    vim kube-controller-manager.yaml
    /k8s-cert 
    # path: /etc/kubernetes/WRONG-PKI-DIRECTORY를 다음과 같이 수정
      path: /etc/kubernetes/pki
    ```

    ```bash
    # 확인
    kubectl get pods -n kube-system
    kubectl get all --all-namespaces
    ```