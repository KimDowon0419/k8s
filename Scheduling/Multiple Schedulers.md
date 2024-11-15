# Multiple Schedulers

## 목차
1. [Multiple Schedulers란 무엇인가?](#Multiple-Schedulers란-무엇인가)
2. [Multiple Schedulers의 목적](#Multiple-Schedulers의-목적)
3. [Multiple Schedulers 구성 방법](#Multiple-Schedulers-구성-방법)
4. [Pod에 사용자 정의 스케줄러 지정](#Pod에-사용자-정의-스케줄러-지정)
5. [Multiple Schedulers 관리 및 운영](#Multiple-Schedulers-관리-및-운영)
6. [Multiple Schedulers 유용한 명령어 모음](#Multiple-Schedulers-유용한-명령어-모음)

---

## Multiple Schedulers란 무엇인가?

**Multiple Schedulers**는 Kubernetes 클러스터에서 **기본 스케줄러 이외에 사용자 정의 스케줄러를 추가로 구성하여 특정 Pod에 다른 스케줄링 정책을 적용**할 수 있도록 하는 기능입니다. Kubernetes는 기본적으로 `kube-scheduler`라는 기본 스케줄러를 사용하지만, 복잡한 스케줄링 요구 사항이 있을 경우 여러 스케줄러를 동시에 운영할 수 있습니다.

- **기능**: 기본 스케줄러 외에 사용자 정의 스케줄러를 통해 다양한 스케줄링 정책을 적용
- **사용 사례**: 특정 워크로드에 대한 커스텀 스케줄링, 다양한 우선순위 정책 적용, 리소스 격리 등

---

## Multiple Schedulers의 목적

Multiple Schedulers를 사용하면 다음과 같은 목적을 달성할 수 있습니다.

1. **특정 워크로드에 맞춘 스케줄링**: 기본 스케줄러로 커버할 수 없는 특정 요구 사항을 만족시키기 위해 사용자 정의 스케줄러를 적용
2. **리소스 최적화**: 특정 노드에 워크로드를 집중시키거나, 특정 조건을 갖춘 노드만 선택하도록 커스텀 스케줄러 구성
3. **클러스터 관리 유연성**: 각 스케줄러에 다양한 정책을 적용하여 클러스터의 리소스 사용을 최적화하고 관리 유연성을 제공

---

## Multiple Schedulers 구성 방법

Multiple Schedulers를 구성하려면 **사용자 정의 스케줄러의 설정 파일**을 작성하고, kube-scheduler와 별도로 실행할 수 있도록 설정합니다. 이 스케줄러는 기본 스케줄러와는 다른 이름으로 정의되며, 지정된 Pod에 대해서만 작동합니다.

### 사용자 정의 스케줄러 YAML 파일 예시

아래 예시는 `custom-scheduler`라는 사용자 정의 스케줄러를 정의한 YAML 파일입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-scheduler
  namespace: kube-system
spec:
  containers:
    - name: kube-scheduler
      image: k8s.gcr.io/kube-scheduler:v1.20.0
      command:
        - kube-scheduler
        - --scheduler-name=custom-scheduler
        - --config=/etc/kubernetes/config/custom-scheduler-config.yaml
      volumeMounts:
        - name: config-volume
          mountPath: /etc/kubernetes/config
  volumes:
    - name: config-volume
      configMap:
        name: custom-scheduler-config
```

1. **scheduler-name 플래그**: `--scheduler-name=custom-scheduler` 플래그를 사용해 스케줄러의 이름을 지정하여 기본 스케줄러와 구분
2. **config**: 특정 스케줄링 설정을 적용하려면 스케줄러의 설정 파일 경로를 명시

---

## Pod에 사용자 정의 스케줄러 지정

Pod를 사용자 정의 스케줄러로 스케줄링하려면 Pod의 `spec.schedulerName` 필드를 설정해 해당 스케줄러가 Pod를 배치하도록 지정합니다.

### 사용자 정의 스케줄러를 사용하는 Pod 예시

아래 예시는 `custom-scheduler`라는 사용자 정의 스케줄러에 의해 스케줄링될 Pod의 YAML 설정입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  schedulerName: custom-scheduler
  containers:
    - name: nginx
      image: nginx
```

1. **schedulerName 필드**: `schedulerName` 필드를 `custom-scheduler`로 지정하여 해당 스케줄러가 Pod를 관리하도록 설정

---

  ```

Multiple Schedulers는 다양한 스케줄링 요구를 충족시키기 위한 유연한 방법을 제공하며, 특정 워크로드에 맞춘 스케줄링 정책을 적용하여 클러스터 관리 효율성을 높일 수 있습니다.
