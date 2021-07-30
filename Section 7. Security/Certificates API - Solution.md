# Certificates API - Solution

# KEYWORD:
kind: CertificateSigningRequest /
kubectl get csr /
kubectl certificate approve /
kubectl certificate deny /
.csr 만드는 방법 (csr 객체 아님)

- akshay.csr 파일 내용이 포함된 이름이 akshay으로 CertificateSigningRequest 객체를 만듭니다.

    ```bash
    kubectl api-versions | grep certif

    cat akshay.csr | base64 | tr -d '\n' > csr.txt
    # tr -d '\n' \n을 모두 삭제

    # 둘이 같은지 확인
    cat akshay.csr
    base64 -d csr.txt
    ```

    ```yaml
    # akshay-csr.yaml

    apiVersion: certificates.k8s.io/v1beta1
    kind: CertificateSigningRequest
    metadata:
      name: akshay
    spec:
      request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXhIR1ZVbnpNU0hlY3U1aWV1L2t2ZE1NYnNTTTNaUk9FNFZldHY3NmFnSzN5CnZXdGJGV29iK3FwZHREaXlnRXFSMUZyY3Z0eFhUdDB3OWdOUlB0RzhHaFRFaWFMSUlBZGRCYXNjUmdVeDVOMjYKbWhaamdJdTNwS0JPbENjeWNWd0ZHc2YzWDliSExua1RWL0ovd3gyRlJ4TGs4Q1N1WHVMNjhQRWRiSGErVUVrTwp4KzBUVGN6aVBoRWFZUlF4OGNWc1hsWGNIRldPZEd6Tk1pWFpjeVh4ejlDRU5ubmcxZHFBQnZveUlOdnk1cnMwCkUyWG1IZ0FKL09haWJOR2pmdWwxNCtHZFhRaThzaGNXWkkwZmRPd0k2SWxIWVl0amk5UCtmeGdZSWUzRlYrdGoKSkFRQTAwZElWSWUwOUtJdXpZNDNsRkVwekxsS2cwOTNEcVB5M1pqY1N3SURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBS080blNINE80ZU9CcEUxKzJZVVY3dENCQzMxUEo5SWhYRGdjM1ozZm9pSm5BS1M1Sk1tCmFSanZiOHNGNkZ5Q0xvNFRWZnFBRkxlUktvM25VNnJGeVE3eEkvK2tBSmxQR2FuVGZ1cHZYaHBkNE5DeWdNK3EKQXhmYmRTR2hiUjZ5OHJBcExyamdiR3F6dzh3YkpNN3FnMzNLRWg0L0NEU1kwTmM3MXpXN3hPODNxMklLdTc5SQpraTBWS0JRSzdBQk1kTTczVlp0Tk9lOHZWdHh2WXM3MFpGcXNYUGFacloweGRQVDFSZkRzamYxMFJiM1M0MWUzCnMzWWYxaU1CdWIzV0RmWlFSbDFZRUFrcVZlMENsRmhaWUxqcEo5K2xEMUpRQUh2eWQ0VlcrM2pDWWJkVVNrRnMKRDRLUzB0QVREcXVBSkFKTU1jbUx2d2RXRi8vbGtsYUVwOUk9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
      usages:
      - digital signature
      - key encipherment
      - server auth

    #  groups:
    #  - system:master 
    #  - system:authenticated
    #  띄어쓰기 없음. 띄어쓰면 오류난다.
    ```

    ```bash
    kubectl create -f akshay-csr.yaml
    kubectl get csr
    ```

    [Manage TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

- 새로 작성된 CertificateSigningReques 객체의 상태는 무엇입니까?

    ```bash
    kubectl get csr

    # Pending
    ```

- CSR 요청 승인

    ```bash
    # 이름 확인
    kubectl get csr

    kubectl certificate approve akshay # 승인
    kubectl certificate deny akshay # 거부
    kubectl get csr
    ```

    [Manage TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

- 몇 개의 csr이 있나?

    ```bash
    kubectl get csr
    ```

- name이 csr-*은 어떤 노드가 요청했나?

    ```bash
    kubectl get csr
    kubectl describe csr csr-5bncs

    # REQUESTOR 부분
    # master node
    ```

- 정기 점검 중에 새로운 CSR 요청이 있음을 깨달았습니다. 이 요청의 이름은 무엇입니까?

    ```bash
    kubectl get csr
    # agent-smith
    ```

- agent-smith는 어느 노드에 csr 요청을 했나? yaml 파일을 확인하시오.

    ```bash
    kubectl get csr agent-smith -o yaml > agent-smith.yaml
    vim agent-smith.yaml

    # system:master
    ```

- agent-smith CSR 요청 거부

    ```bash
    kubectl get csr
    kubectl certificate deny agent-smith
    ```

- agent-smith CSR 삭제

    ```bash
    kubectl delete csr agent-smith
    ```