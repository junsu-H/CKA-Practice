# Monitor Application Logs - Solution

# KEYWORD:
kubectl logs webapp-2 -c simple-webapp

- webapp-1 user5의 로그인 실패 로그 확인 ( web pod 1개 )

    ```bash
    kubectl get pods
    # webapp-1

    kubectl describe pods webapp-1

    kubectl logs webapp-1 | grep -i user5
    ```

- webapp-2 살펴보기 ( web + db라서 pod가 2개 )

    ```bash
    kubectl get pods 
    kubectl describe pods webapp-2 | grep -i -A10 event
    # create container simple-webapp

    kubectl logs webapp-2 -c simple-webapp
    # 2개일 때 컨테이너 지정해줘야 한다.
    ```

    ![Untitled 1](https://user-images.githubusercontent.com/63388678/104089615-a83b1500-52b3-11eb-81d2-d0f9bc10fb60.png)

- webapp-2 user 로그인 문제 찾기

    ```bash
    kubectl logs webapp-2 -c simple-webapp | grep -i user1
    kubectl logs webapp-2 -c simple-webapp | grep -i user2
    kubectl logs webapp-2 -c simple-webapp | grep -i user4
    kubectl logs webapp-2 -c simple-webapp | grep -i user30
    ```