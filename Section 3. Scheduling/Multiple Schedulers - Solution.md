# Multiple Schedulers - Solution

- 쿠버네티스 스케줄러를 배포하는 pod의 이름은?

    ```bash
    kubectl get pods -o wide --all-namespaces | grep -i scheduler

    # kube-scheduler-master
    ```

- 스케줄러를 배포하는데 사용하는 이미지는?

    ```bash
    ps -aux | grep config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests
    ls 
    cat kube-scheduler.yaml | grep -i image

    # 또는
    kubectl describe pods kube-scheduler-master -n kube-system | grep -i -A10 -B10 image 
    ```

- 지정된 스펙에 따라 추가 스케줄러를 클러스터에 배치하십시오.
    - Namespace: kube-system
    - Name: my-scheduler
    - Status: Running
    - Custom Scheduler Name

    ```bash
    cd /etc/systemd/system/kubelet.service.d/
    ls
    cat 10-kubeadm.config | grep -i kubelet
    # /var/lib/kubelet/config.yaml

    # 또는
    ps -aux | grep -i config
    # /var/lib/kubelet/config.yaml

    cat /var/libe/kubelet/config.yaml | grep -i static
    # /etc/kubernetes/manifests/

    cd /etc/kubernetes/manifests/
    ls
    cp kube-scheduler.yaml ../my-scheduler.yaml
    ```

    ```yaml
    # cat /etc/kubernetes/manifests/kube-scheduler.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:    
        component: kube-scheduler
        tier: control-plane
      name: kube-scheduler
      namespace: kube-system
    spec:
      containers:
      - command:
        - kube-scheduler
        - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
        - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
        - --bind-address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=true
        image: k8s.gcr.io/kube-scheduler:v1.16.0
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 8
          httpGet:
            host: 127.0.0.1
            path: /healthz
            port: 10251
            scheme: HTTP
          initialDelaySeconds: 15
          timeoutSeconds: 15
        name: kube-scheduler
        resources:
          requests:
            cpu: 100m
        volumeMounts:
        - mountPath: /etc/kubernetes/scheduler.conf
          name: kubeconfig
          readOnly: true
      hostNetwork: true
      priorityClassName: system-cluster-critical
      volumes:
      - hostPath:
          path: /etc/kubernetes/scheduler.conf
          type: FileOrCreate
        name: kubeconfig
    status: {}
    ```

    ```yaml
    # vim /etc/kubernetes/manifests/my-scheduler.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: my-scheduler
      namespace: kube-system
    spec:
      containers:
      - name: my-scheduler-con
        image: k8s.gcr.io/kube-scheduler-amd64:v1.16.0
        command:
        - kube-scheduler
        - --address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=false
        - --scheduler-name=my-scheduler
    #   - --port=10282
    #   - --secure-port=0

    # leader-elect=false
    # 기본값 true
    # 메인 루프를 실행하기 전에 리더 선거 클라이언트를 시작하고 
    # 리더십을 확보하십시오. 
    # 고 가용성을 위해 복제 된 구성 요소를 실행할 때 기능을 활성화하십시오.

    # secure-port=0 
    # 인증 및 권한 부여로 HTTPS를 제공 할 포트입니다. 
    # 0이면 HTTPS를 전혀 제공하지 않습니다.

    # /etc/kubernetes/manifests/kube-scheduler.yaml를 보고 
    # 위와 같이 수정

    # --port는 netstat -ano로 안 쓰는 port 확인
    ```

    [https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

    [Configure Multiple Schedulers](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)

- /root/nginx-pod.yaml 파일을 이용해 아래 조건에 맞게 pod를 작성하시오
    - Name: nginx
    - Uses custom scheduler
    - Status: Running

    ```yaml
    # nginx-pod.yaml 

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      -  image: nginx
         name: nginx
      schedulerName: my-scheduler

    # kubectl describe pods nginx | grep -i node 
    # nodeName은 node가 할당이 안 되어 있으면 추가.
    ```

    ```bash
    kubectl create -f nginx-pod.yaml
    kubectl get events | grep -i my-scheduler
    ```

    [Configure Multiple Schedulers](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)