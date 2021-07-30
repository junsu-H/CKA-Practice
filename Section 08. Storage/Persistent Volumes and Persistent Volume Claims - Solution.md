# Persistent Volumes and Persistent Volume Claims - Solution

# KEYWORD:
kubectl get pv /
kubectl get pvc /
volumeMounts: /
volumes: /
ReadWriteOnce / 
ReadOnlyMany /
ReadWriteMany

- 응용 프로그램은 /log/app.log 위치에 로그를 저장합니다. 로그를보십시오.

    ```bash
    kubectl get pods
    kubectl exec -it webapp cat /log/app.log
    ```

- POD를 지금 삭제한 경우 이러한 로그를 볼 수 있습니까?

    ```bash
    kubectl delete pods webapp
    kubectl exec -it webapp cat /log/app.log

    # 불가능
    ```

- 호스트의 /var/log/webapp에 이러한 로그를 저장하도록 볼륨을 구성하십시오. 주어진 사양을 사용하십시오.
    - Name: webapp
    - Image Name: kodekloud/event-simulator
    - Volume HostPath: /var/log/webapp
    - Volume Mount: /log

    ```yaml
    # webapp-pod-hostpath.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp
    spec:
      containers:
      - image: kodekloud/event-simulator
        name: webapp-con
        volumeMounts:
        - mountPath: /log # container의 경로
          name: log-volume
      volumes:
      - name: log-volume #  같아야 한다.
        hostPath:
          path: /var/log/webapp # container와 연결될 node의 경로
    #     type: Directory # 해당 노드에 /var/log/webapp가 존재해야 사용 가능

    # type에는 4개가 있다.
    # type: Directory # 디렉터리가 미리 존재해야 한다.
    # type: DirectoryOrCreate # 디렉터리가 없으면 생성한다.
    # type: File # 파일이 미리 존재해야 한다.
    # type: FileOrCreate # 파일이 없으면 생성한다.
    ```

    [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

```bash
ReadWriteOnce 단일 노드만 읽기/쓰기를 위해 볼륨을 마운트할 수 있다.
ReadOnlyMany 여러 노드가 읽기 위해 볼륨을 마운트할 수 있다.
ReadWriteMany 여러 노드가 읽기/쓰기를 위해 볼륨을 마운트할 수 있다.
```

- 주어진 사양으로 PV를 만듭니다.
    - Volume Name: pv-log
    - Storage: 100Mi
    - Access modes: ReadWriteMany
    - Host Path: /pv/log

    ```yaml
    # pv-log.yaml

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-log
    spec:
      storageClassName: manual # kind: StorageClass임, 현재는 없어도 됨.
      capacity:
        storage: 100Mi
      accessModes:
        - ReadWriteMany
      hostPath:
        path: /pv/log # 노드에 /pv/log가 있어야 한다.
        type: DirectoryOrCreate

    # type에는 4개가 있다.
    # type: Directory # 디렉터리가 미리 존재해야 한다.
    # type: DirectoryOrCreate # 디렉터리가 없으면 생성한다.
    # type: File
    # type: FileOrCreate
    ```

    [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

- 주어진 사양으로 PVC 만듭니다.
    - Volume Name: claim-log-1
    - Storage Request: 50Mi
    - Access modes: ReadWriteOnce

    ```yaml
    # claim-log-1.yaml

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: claim-log-1
    spec:
      storageClassName: manual # kind: StorageClass임, 현재는 없어도 됨.
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 50Mi
    ```

    [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

- PVC 상태는 무엇입니까?

    ```bash
    kubectl get pvc

    # Pending
    ```

- PV 상태는 무엇입니까?

    ```bash
    kubectl get pv

    # Available
    ```

- PVC가 사용 가능한 PV에 속하지 않는 이유는 무엇입니까?

    ```bash
    PV와 PVC에 설정된 액세스 모드가 일치하지 않습니다
    ```

- 클레임의 액세스 모드를 업데이트하여 PVC에 바인딩합니다. claim-log-1 삭제 및 재 작성
    - Volume Name: claim-log-1
    - Storage Request: 50Mi
    - PVol: pv-log
    - Status: Bound

    ```yaml
    # claim-log-1.yaml

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: claim-log-1
    spec:
      storageClassName: manual # kind: StorageClass임, 현재는 없어도 됨.
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 50Mi
    ```

- PVC에 사용할 수있는 용량은 50Mi입니까?

    ```bash
    kubectl get pvc

    # CAPACITY 100Mi
    ```

- PVC 스토리지로 사용하도록 webapp 포드를 업데이트하십시오. 이전에 구성된 hostPath를 새로 작성된 PersistentVolumeClaim으로 바꾸십시오.
    - Name: webapp
    - Image Name: kodekloud/event-simulator
    - Volume: PersistentVolumeClaim=claim-log-1
    - Volume Mount: /log

    ```yaml
    # webapp-pod-pvc.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp
    spec:
        containers:
        - name: webapp-pod
          image: kodekloud/event-simulator
          volumeMounts:
          - mountPath: /log
            name: log-volume
        volumes:
        - name: log-volume
          persistentVolumeClaim:
            claimName: claim-log-1
    ```

    [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#persistentvolumeclaim)

- PV에 설정된 Reclaim Policy - 'pv-log'

    ```bash
    kubectl get pv | grep -i -A3 recl

    # Retain
    ```

- PVC가 파손되면 PV는 어떻게됩니까?

    ```bash
    PV는 삭제되지 않지만 사용할 수 없다.
    ```

- PVC를 삭제하면 어떻게 되는가?

    ```bash
    kubectl get pvc
    kubectl delete pvc claim-log-1

    # PVC가 termination 상태로 된다.
    ```

- PVC가 termination 상태인 이유는 무엇입니까?

    ```bash
    Pod에서 PVC를 사용 중이다.
    ```

- pod webapp 삭제

    ```bash
    kubectl delete pods webapp
    ```

- 현재 PVC 상태는 어떻습니까?

    ```bash
    kubectl get pvc

    # 삭제됨.
    # pod가 삭제되면 pvc도 같이 삭제된다.
    ```

- 현재 PV의 상태는 무엇입니까?

    ```bash
    kubectl get pv | grep -i -A3 STATUS

    # Released
    # pod가 삭제돼도 삭제되지 않는다.
    ```