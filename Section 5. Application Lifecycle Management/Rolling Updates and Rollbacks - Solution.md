# Rolling Updates and Rollbacks - Solution

# KEYWORD:
RollingUpdate /
Recreate

```bash
# deploy 생성
kubectl create deploy frontend --image=kodekloud/webapp-color:v2

# 롤업데이트 로그 남기기
kubectl set image deploy frontend webapp-color=kodekloud/webapp-color:v3 --record

# 업데이트 로그
kubectl rollout history deploy frontend

# 롤아웃 되돌리기
kubectl rollout undo deploy frontend

# 특정 디플로이먼트 버전으로 돌리기
kubectl rollout undo deploy frontend --to-revision=1
```

- 웹 애플리케이션의 색깔?

    Quiz Portal 옆에 Webapp Portal 클릭하면 됨.

    ```bash
    kubectl get pods
    kubectl get deploy
    kubectl get rs
    kubectl get svc
    ```

- pod 개수

    ```bash
    kubectl get pods
    ```

- deployment 개수, 사용한 이미지

    ```bash
    kubectl get deploy
    kubectl describe deploy | grep -i image
    ```

- deployment의 현재 상황

    ```bash
    kubectl describe deploy | grep -i strategy

    # RollingUpdate
    ```

- 지금 응용 프로그램을 업그레이드 할 경우 어떻게 될까?

    ```bash
    RollingUpdate는 pod를 n개씩 업그레이드 함

    cf) Recreate는 pod를 모두 삭제 후 재생성
    ```

- deployment 이미지를 kodekloud/webapp-color:v2로 업데이트

    ```bash
    kubectl set image deploy frontend \
      simple-webapp=kodekloud/webapp-color:v2 \
      --record

    # 또는
    kubectl get deploy 
    kubectl edit deploy frontend
    /image
    kodekloud/webapp-color:v1 ---> kodekloud/webapp-color:v2 수정

    ```

- 한 번에 업데이트할 수 있는 pod 수

    ```bash
    kubectl get all
    kubectl describe deploy frontend
    kubectl describe deploy frontend | grep -i -A10 -B10 max
    # 25% 나옴
    # Replicas가 4이므로 4의 25%는 1
    ```

- 배포 전략을 '다시 만들기'로 변경
    - Deployment Name: frontend
    - Deployment Image: kodekloud/webapp-color:v2
    - Strategy: Recreate

    ```bash
    kubectl get all
    kubectl describe deploy frontend
    kubectl edit deploy frontend
    # rollingUpdate:
    #   maxSurge: 25%
    #   maxUnavailable: 25% 
    # 세 개 삭제
    # type: Recreate
    ```

    [디플로이먼트](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/#%eb%94%94%ed%94%8c%eb%a1%9c%ec%9d%b4%eb%a8%bc%ed%8a%b8-%eb%a1%a4%eb%b0%b1)

- deployment 이미지를 kodekloud/webapp-color:v3로 업데이트

    ```bash
    kubectl set image deploy frontend \
      webapp-color=kodekloud/webapp-color:v3 \
      --record

    # 또는
    kubectl get deploy 
    kubectl edit deploy frontend
    /image
    kodekloud/webapp-color:v2 ---> kodekloud/webapp-color:v3 수정
    ```

- 업데이트 후 running 상태 확인

    ```bash
    kubectl get deploy
    kubectl get pods
    ```

```bash
#!/bin/bash
# curl-test.sh

for i in {1..35}; do
   kubectl exec --namespace=kube-public curl -- sh -c 'test=`wget -qO- -T 2  http
://webapp-service.default.svc.cluster.local:8080/info 2>&1` && echo "$test OK" ||
 echo "Failed"';
   echo ""
```