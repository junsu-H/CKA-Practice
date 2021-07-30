# Explore CNI Weave - Solution

# KEYWARD:
/opt/cni/bin / 
/etc/cni/net.d/

- kubelet 서비스를 검사하고 Kubernetes에 대해 구성된 네트워크 플러그인을 식별하십시오.

    ```bash
    ps -aux | grep -i kubelet | grep -i plugin

    # -ax 모든 프로세스 표시
    # -u 특정 사용자 프로세스를 보여준다.
    ```

- 모든 바이너리의 CNI 지원 플러그인으로 구성된 경로는 무엇입니까?

    ```bash
    ps -aux | grep -i cni | grep -i dir

    # --cni-bin-dir
    ```

- 이 호스트에서 사용 가능한 CNI 플러그인 목록에서 사용할 수없는 플러그인을 식별하십시오.

    ```bash
    # kubectl describe all -n kube-system | grep -i -A3 args

    cd /opt/cni/bin
    ls -al

    # cisco
    ```

- 이 kubernetes 클러스터에서 사용하도록 구성된 CNI 플러그인은 무엇입니까?

    ```bash
    ls /etc/cni/net.d/
    cat 10-weave.conflist

    # /etc/cni/net.d/ default 경로임.
    # weave
    ```

    [Network Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

- 컨테이너와 관련 네임 스페이스가 생성 된 후 kubelet이 실행할 바이너리 실행 파일

    ```bash
    cat /etc/cni/net.d/10-weave.conflist | grep -i -A5 type

    # weave-net
    ```