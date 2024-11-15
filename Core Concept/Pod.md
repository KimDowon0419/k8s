Pod
====


```yaml
kubectl run redis --image = redis123 --dry-run=client -o yaml
```
pod 생성하는 yaml을 생성

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
```
```yaml
kubectl apply -f sample.yaml
```
