# Advanced Kubectl Commands - Solution

# KEYWORD:
-o json > /
jsonpath /
kubectl config view --

- JSON 형식의 노드 목록을 가져와서 `/opt/outputs/nodes.json`에 저장하시오.

    ```bash
    kubectl get nodes -o json > /opt/outputs/nodes.json
    ```

- `node01` 세부사항을 json 형식으로 가져와서 `/opt/outputs/node01.json`에 저장하시오.

    ```bash
    kubectl get nodes
    kubectl get nodes node01 -o json > /opt/outputs/node01.json
    ```

- JSON PATH 쿼리를 사용하여 노드 이름만 가져와서 `/opt/outputs/node_names.txt` 에 저장하시오.

    ```bash
    kubectl edit nodes # jsonpath 경로 확인
    kubectl get nodes -o jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt
    ```

- JSON PATH 쿼리를 사용하여 모든 노드의 `osImage` 를 가져와서  `/opt/outputs/nodes_os.txt` 파일에 저장하시오. `osImage` 의 경로는 `status.nodeInfo.osImage`

    ```bash
    kubectl edit nodes # jsonpath 확인
    kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt
    ```

- `/root/my-kube-config`에 kube-config가 있다. 여기서 사용자 이름을 가져와서 `/opt/outputs/users.txt` 에 저장하시오. kube-config view는 다음 명령어를 사용한다. 
`kubectl config view --kubeconfig=/root/my-kube-config`

    ```bash
    kubectl config view --kubeconfig=/root/my-kube-config
    kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.users[*].name}" > /opt/outputs/users.txt

    # jsonpath 감싸는 건 큰 따옴표든 작은 따옴표든 상관 없음
    ```

- PV를 용량기준으로 정렬하고 해당 결과를 `/opt/outputs/storage-capacity-sorted.txt`에 저장하시오.

    ```bash
    kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt
    ```

    [kubectl 치트 시트](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/)

- 위에서 했던 PV를 2개의 컬럼만 **검색하여 `/opt/outputs/pv-and-capacity-sorted.txt` 하시오.
2개의 컬럼은 각각 `NAME`과 `CAPACITY`이다. `custom-columns`옵션을 사용하십시오 . 
그리고 위 질문과 동일하게 정렬해야 한다.**

    ```bash
    kubectl get pv --sort-by=.spec.capacity.storage -o \
        custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage \
        > /opt/outputs/pv-and-capacity-sorted.txt
    ```

    [kubectl 치트 시트](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/)

- **JSON PATH 쿼리를 통해** `my-kube-config` **컨텍스트 파일에서** `aws-user`**로 구성된 컨텍스트를 식별하고 결과를** `/opt/outputs/aws-context-name`**에 저장하시오.**

    ```bash
    kubectl config view --kubeconfig=/root/my-kube-config

    kubectl config view --kubeconfig=/root/my-kube-config \
        -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" \
        > /opt/outputs/aws-context-name
    ```

    [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)