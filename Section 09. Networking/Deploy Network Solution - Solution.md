# Deploy Network Solution - Solution

# KEYWORD:
watch "kubectl get pods" 또는
kubectl get pods -w

- 이 실습 테스트에서는 `weave-net`POD 네트워킹 솔루션을 클러스터에 설치 합니다. 먼저 설정을 검사하겠습니다. 클러스터에 배포 된 포드의 상태는 무엇입니까?

    ```bash
    kubectl get pods

    # NotRunning
    ```

- POD가 실행되지 않는 이유 검사

    ```bash
    kubectl describe pods app-9d866df94-mjh68

    # no networking
    ```

- weave-net클러스터에 네트워킹 솔루션 배포

    ```bash
    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    watch "kubectl get pods"
    ```

    [Creating a single control-plane cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)