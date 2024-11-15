# Static Pods

## 목차
1. [Static Pods란 무엇인가?](#Static-Pods란-무엇인가)
2. [Static Pods의 목적](#Static-Pods의-목적)
3. [Static Pods 설정 방법](#Static-Pods-설정-방법)
4. [Static Pods 관리 및 운영](#Static-Pods-관리-및-운영)
5. [Static Pods 유용한 명령어 모음](#Static-Pods-유용한-명령어-모음)

---

## Static Pods란 무엇인가?

**Static Pods**는 **Kubernetes 스케줄러를 사용하지 않고, 특정 노드에서 kubelet에 의해 직접 관리되는 Pod**입니다. Static Pod는 노드의 kubelet이 시작할 때, 지정된 디렉토리에서 YAML 또는 JSON 파일을 찾아 자동으로 실행합니다. 이는 주로 Kubernetes 클러스터의 핵심 컴포넌트를 관리할 때 사용됩니다.

- **특징**: 스케줄러 없이 노드 단위에서 관리되며, kubelet에 의해 시작되고 유지 관리
- **사용 위치**: 일반적으로 **control plane 노드**에서 실행
- **목적**: API 서버, 컨트롤러 매니저 등 클러스터의 핵심 서비스에 필요한 Pod 관리

---

## Static Pods의 목적

Static Pods는 Kubernetes의 중요한 컴포넌트와 함께 작동하며, 클러스터가 정상적으로 작동하는 데 필요한 핵심 Pod를 관리합니다.

1. **클러스터 핵심 컴포넌트 관리**: API 서버, 컨트롤러 매니저 등 클러스터의 핵심 서비스 Pod를 control plane 노드에서 실행할 때 사용
2. **스케줄러 의존성 감소**: Kubernetes 스케줄러를 사용하지 않고 각 노드에서 kubelet이 직접 관리하므로 스케줄러에 의존하지 않음
3. **단순 관리**: kubelet이 Static Pod의 설정 파일을 직접 읽어 실행하므로, 설정 파일 수정 시 자동으로 업데이트

---

## Static Pods 설정 방법

Static Pods는 노드의 kubelet이 **지정된 경로에 있는 설정 파일**을 읽고 자동으로 실행합니다. 이 경로는 kubelet 설정에 따라 다를 수 있지만, 일반적으로 `/etc/kubernetes/manifests` 디렉토리가 사용됩니다.

### Static Pod 설정 예시

아래는 `kube-apiserver` Static Pod를 설정하는 YAML 파일 예시입니다. 이 파일을 `/etc/kubernetes/manifests/kube-apiserver.yaml`에 저장하면 kubelet이 자동으로 실행합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
    - name: kube-apiserver
      image: k8s.gcr.io/kube-apiserver:v1.20.0
      command:
        - kube-apiserver
        - --advertise-address=192.168.0.1
        - --allow-privileged=true
      ports:
        - containerPort: 6443
          name: https
```

1. **YAML 파일 위치**: `/etc/kubernetes/manifests` 디렉토리에 YAML 파일을 배치
2. **kubelet이 자동 실행**: kubelet이 지정된 디렉토리를 모니터링하여 파일을 읽고 Pod를 자동으로 생성
3. **설정 업데이트**: YAML 파일을 수정하면 kubelet이 자동으로 변경 사항을 반영하여 Static Pod를 업데이트

---

## Static Pods 관리 및 운영

Static Pods는 Kubernetes API를 통해 직접 관리되지 않기 때문에 일부 제한 사항이 있으며, kubelet이 직접 Pod를 생성하고 관리합니다.

1. **Static Pods 상태 확인**
   - Static Pod는 일반 Pod와 같이 `kubectl get pods -n kube-system` 명령어로 확인할 수 있습니다.
   ```bash
   kubectl get pods -n kube-system
   ```

2. **Static Pod 설정 파일 업데이트**
   - 설정 파일을 수정하면 kubelet이 자동으로 변경 사항을 반영하여 Static Pod를 업데이트합니다.
   ```bash
   vi /etc/kubernetes/manifests/kube-apiserver.yaml
   ```

3. **Static Pod 삭제**
   - Static Pod를 삭제하려면 해당 YAML 파일을 디렉토리에서 제거해야 합니다.
   ```bash
   rm /etc/kubernetes/manifests/kube-apiserver.yaml
   ```

4. **Static Pod 로그 확인**
   - 일반 Pod와 동일하게 `kubectl logs` 명령어를 통해 로그를 확인할 수 있습니다.
   ```bash
   kubectl logs -n kube-system kube-apiserver
   ```

---

## Static Pods 유용한 명령어 모음

- **Static Pod 상태 확인**
  ```bash
  kubectl get pods -n kube-system
  ```

- **Static Pod 로그 확인**
  ```bash
  kubectl logs -n kube-system <pod-name>
  ```

- **Static Pod 설정 파일 수정**
  ```bash
  vi /etc/kubernetes/manifests/<static-pod-name>.yaml
  ```

- **Static Pod 삭제**
  ```bash
  rm /etc/kubernetes/manifests/<static-pod-name>.yaml
  ```

Static Pods는 kubelet에 의해 직접 관리되므로 클러스터 핵심 컴포넌트를 안정적으로 유지할 수 있습니다. Kubernetes 스케줄러가 필요하지 않은 클러스터의 중요한 Pod를 제어하고 관리하는 데 매우 유용합니다.
