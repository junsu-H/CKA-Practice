# Static Pods - Solution

# KEYWORD:
ps -aux | grep -i kubelet / 
cat /var/lib/kubelet/config.yaml  | grep -i static

- static pod의 개수는?

    ```bash
    # staticPod 위치

    ps -aux | grep kubelet | grep config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml  | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests/
    ls | wc -l

    # 또는
    kubectl get pods --all-namespaces | grep -i master
    ```

- static pod로 배포되지 않은 것은?

    ```bash
    kubectl get pods --all-namespaces | grep -i -v master

    # -v 해당 문자열 제외
    ```

- static pod로 배포되지 않은 것은?

    ```bash
    kubectl get pods --all-namespaces | grep -i -v master

    # -v 제외
    # kube-proxy가 답
    ```

- static pod는 어디 노드에 만들어졌나?

    ```bash
    kubectl get pods --all-namespaces -o wide

    # master
    ```

- static pod의 디렉터리 경로는?

    ```bash
    ps -aux | grep kubelet # 모든 프로세스를 보여준다

    # --config=/var/lib/kubelet/config.yaml 확인해서

    cat /var/lib/kubelet/config.yaml | grep -i path
    /etc/kubernetes/manifest
    ```

- /etc/kubernetes/manifests에는 몇 개의 파일이 있는가?

    ```bash
    vim /etc/kubernetes/manifests

    # 또는
    cd /etc/kubernetes/manifests
    ls
    ```

- kube-apiserver를 static pod를 배포하는데 사용하는 docker 이미지는?

    ```bash
    cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i image
    ```

- 다음을 만족하는 static-pod를 만드시오.
    - Name: static-busybox
    - Image: busybox
    - command: sleep 1000

    ```bash
    ps -aux | grep kubelet | grep config
    # --config=/var/lib/kubelet/config.yaml

    cat /var/lib/kubelet/config.yaml  | grep -i static
    # staticPodPath: /etc/kubernetes/manifests

    cd /etc/kubernetes/manifests/
    vim busybox-static_pod.yaml
    ```

    ```yaml
    # busybox-static_pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: static-busybox
    spec:
      containers:
      - name: static-busybox
        image: busybox
        command: ["sleep 1000"]
    ```

    ---

    ```bash
    kubectl run --generator=run-pod/v1 static-busybox \
        --image=busybox \
        --command -- sleep 1000 \
        --dry-run -o yaml \
        > /etc/kubernetes/manifests/static-busybox.yaml
    ```

    ```yaml
    # static-busybox.yaml

    apiVersion: v1                                                           skind: Pod                                                                       s
    metadata:
      labels:
        run: static-busybox
      name: static-busybox
    spec:
      containers:
      - command:
        - "sleep"
        - "1000"
        image: busybox
        name: static-busybox
    ```

- busybox:1.28.4로 image를 변경하시오.

    ```bash
    vim /etc/kubernetes/manifests/busybox-static-pod.yaml
    ```

    ```yaml
    # busybox-static-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: static-busybox
    spec:
      containers:
      - name: static-busybox
        image: busybox:1.28.4
        command: ["sleep", "1000"]

    :wq!
    ```

    ```bash
    kubectl create -f busybox-static-pod.yaml
    ```

- pod 이름이 static-greenbox-node01인 static pod를 삭제하시오.

    ```bash
    # pod 확인
    kubectl get pods

    # node 확인
    kubectl describe pods static-greenbox-node01 | grep -i node

    # ip 확인
    kubectl get nodes -o wide

    # node01과 ssh로 연결, 172.17.0.32는 node01 ip, ssh node01는 오류난다.
    ssh 172.17.0.32  

    ps -aux | grep -i config
    cat /var/lib/kubelet/config.yaml | grep -i static
    # staticPodPath: /etc/just-to-mess-with-you

    cd /etc/just-to-mess-with-you
    rm -rf greenbox.yaml
    ```