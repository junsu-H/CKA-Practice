# Manual Scheduling - Solution

# KEYWORD:
kubectl logs /
kubectl get events

- pod 생성

    ```yaml
    # nginx.ymal

    apiVersion: v1
    kind: Pod
    metadata:
    	name: nginx

    spec:
    	containers:
    	- name: nginx
    		image: nginx
    ```

    ```bash
    kubectl create -f nginx.yaml

    # 또는
    kubectl run --generator=run-pod/v1 nginx --image=nginx
    ```

- pod 상태 확인

    ```bash
    kubectl get pods
    ```

- pod status가 pending인 이유 트러블 슈팅 방법

    ```bash
    kubectl get pods
    kubectl describe pods nginx # Events에 아무 것도 없음.

    # 위 명령어 실행시 Events에 아무 것도 없을 때, 확인
    kubectl get pods --all-namespaces
    # kube-system에서 오류나는 파드 확인

    kubectl describe pods coredns-5644d7b6d9-22tv4 -n kube-system

    # 오류나는 파드가 안 보이면
    kubectl describe pods -n kube-system | grep -i warning
    # FailedScheduling  <unknown>  default-scheduler
    # 0/1 nodes are available: 1 node(s) had taints 
    # that the pod didn't tolerate.

    # 결론은 할당된 Node가 없다.
    ```

    ![Untitled 1](https://user-images.githubusercontent.com/63388678/104253815-c45bd380-54b8-11eb-8661-48b4903d4097.png)
    
    ![Untitled 2](https://user-images.githubusercontent.com/63388678/104253818-c4f46a00-54b8-11eb-81a4-bc33e4794a00.png)


    [Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

    ```bash
    kubectl get pods --all-namespaces -o wide
    # nginx에 할당된 NODE가 없다.

    kubectl logs nginx
    kubectl get events --all-namespaces | grep -i warning

    # no scheduler present
    ```

- nodeName에 node01 할당

    ```yaml
    # nginx.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
      nodeName: node01
    ```

    ```bash
    kubectl get pods
    kubectl delete pods nginx

    # kubectl apply -f nginx.yaml은 불가능, 노드 위치가 달라서 에러
    kubectl create -f nginx.yaml

    # 입력시 아래와 같이 나옴.
    kubectl describe nodes node01
    ```

    ![Untitled 3](https://user-images.githubusercontent.com/63388678/104253819-c58d0080-54b8-11eb-94a1-110937859e8d.png)

- nodeName에 master 할당

    ```yaml
    # nginx.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
      **nodeName: master**
    ```

    ```bash
    kubectl get pods
    kubectl delete pods nginx

    # kubectl apply -f nginx.yaml은 불가능, 노드 위치가 달라서 에러
    kubectl create -f nginx.yaml
    ```