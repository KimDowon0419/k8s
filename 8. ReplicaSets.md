ReplicaSets
=====================

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:1.21.1  # 변경할 이미지 이름 및 태그로 수정
```
templates의 label과 selector의 matchLabels가 일치하여 인식하여 만들어준다


```shell
kubectl scale --replicas=5 replicaset new-replica-set
```
replica수를 변경하는 명령


