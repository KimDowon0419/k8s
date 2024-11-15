# Imperative Command

## 목차
1. [Imperative Command란 무엇인가?](#Imperative-Command란-무엇인가)
2. [Imperative Command의 특징](#Imperative-Command의-특징)
3. [Imperative Command와 Declarative 방식의 차이점](#Imperative-Command와-Declarative-방식의-차이점)
4. [주요 Imperative Command 예시](#주요-Imperative-Command-예시)
5. [Imperative Command의 장단점](#Imperative-Command의-장단점)
6. [Imperative Command 유용한 명령어 모음](#Imperative-Command-유용한-명령어-모음)

---

## Imperative Command란 무엇인가?

**Imperative Command**는 Kubernetes 리소스를 명령어 한 줄로 즉시 생성, 수정 또는 삭제하는 방식입니다. YAML 파일을 사용하지 않고 **kubectl** 명령어만으로 리소스를 정의하고 적용할 수 있습니다. 이 방법은 빠르게 작업을 수행해야 할 때 유용하며, 특히 테스트나 디버깅 상황에서 많이 사용됩니다.

- **즉각적 실행**: 한 줄 명령어로 Kubernetes 리소스를 빠르게 생성하고 변경
- **테스트와 디버깅에 유용**: 작은 테스트나 간단한 리소스를 실험할 때 적합
- **명령어 기반**: YAML 파일 없이도 Kubernetes 리소스의 주요 설정 가능

---

## Imperative Command의 특징

Imperative Command는 즉각적으로 명령을 실행하는 특성이 있으며, 각 명령어를 통해 리소스를 직접 제어합니다. 이 방식은 **파일을 생성하지 않아도 되는 간단한 작업**에 적합하지만, **리소스의 상태를 기록**하고 **장기적으로 관리**해야 하는 경우에는 권장되지 않습니다.

1. **빠르고 간단한 리소스 생성**: YAML 파일을 작성하지 않고 명령어로 바로 리소스 생성
2. **실험적이고 단기적인 작업에 적합**: 배포나 롤백 기록이 필요 없는 경우 유용
3. **명령어 기반 구성**: 각 리소스를 한 줄 명령어로 구성 가능

---

## Imperative Command와 Declarative 방식의 차이점

| 비교 항목      | Imperative Command                     | Declarative 방식                                 |
|----------------|---------------------------------------|------------------------------------------------|
| 리소스 정의    | 명령어로 직접 정의                      | YAML 파일에 정의 후 `kubectl apply`로 적용      |
| 실행 시점      | 즉시 실행                              | 파일을 통해 반복적으로 실행 가능                |
| 기록 관리      | 변경 사항 기록 어려움                   | git과 같은 버전 관리로 변경 사항 기록 가능      |
| 적합한 상황    | 빠르고 단순한 작업                     | 장기적으로 관리해야 하는 리소스                |
| 유지 보수      | 유지보수 및 자동화 어려움              | YAML 파일로 재생 가능, CI/CD 파이프라인과 연동 |

---

## 주요 Imperative Command 예시

1. **Pod 생성**:
   ```bash
   kubectl run my-pod --image=nginx --restart=Never
   ```

2. **Deployment 생성**:
   ```bash
   kubectl create deployment my-deployment --image=nginx
   ```

3. **Service 생성**:
   ```bash
   kubectl expose deployment my-deployment --port=80 --target-port=80 --type=ClusterIP
   ```

4. **Namespace 생성**:
   ```bash
   kubectl create namespace my-namespace
   ```

5. **ConfigMap 생성**:
   ```bash
   kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
   ```

6. **Secret 생성**:
   ```bash
   kubectl create secret generic my-secret --from-literal=username=myuser --from-literal=password=mypassword
   ```

7. **Scale (스케일링)**:
   ```bash
   kubectl scale deployment my-deployment --replicas=3
   ```

8. **롤백 수행**:
   ```bash
   kubectl rollout undo deployment my-deployment
   ```

9. **Label 추가**:
   ```bash
   kubectl label pod my-pod app=my-app
   ```

10. **Annotation 추가**:
    ```bash
    kubectl annotate pod my-pod description="test pod"
    ```

---

## Imperative Command의 장단점

### 장점
- **빠르고 간편함**: YAML 파일을 작성하지 않고도 Kubernetes 리소스를 즉시 정의하고 배포 가능
- **테스트와 디버깅에 유용**: 리소스 생성과 삭제가 간단하여 실험적 환경에서 유용
- **직관적**: 각 명령어로 리소스를 바로 제어할 수 있어 사용이 간편함

### 단점
- **재사용성 낮음**: 명령어 기반이므로 동일 작업을 반복적으로 실행하기 어려움
- **기록 관리 어려움**: 리소스 변경 사항을 기록하기 어려우며, 배포 이력이 남지 않음
- **대규모 관리에 비효율적**: 여러 리소스를 장기적으로 관리하는 경우, YAML 파일 기반 관리가 더 효율적임

---

## Imperative Command 유용한 명령어 모음

- **Pod 생성**:
  ```bash
  kubectl run my-pod --image=nginx --restart=Never
  ```

- **Deployment 생성**:
  ```bash
  kubectl create deployment my-deployment --image=nginx
  ```

- **Service 생성**:
  ```bash
  kubectl expose deployment my-deployment --port=80 --target-port=80 --type=ClusterIP
  ```

- **Namespace 생성**:
  ```bash
  kubectl create namespace my-namespace
  ```

- **ConfigMap 생성**:
  ```bash
  kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
  ```

- **Secret 생성**:
  ```bash
  kubectl create secret generic my-secret --from-literal=username=myuser --from-literal=password=mypassword
  ```

- **Deployment 스케일링**:
  ```bash
  kubectl scale deployment my-deployment --replicas=3
  ```

- **롤백 수행**:
  ```bash
  kubectl rollout undo deployment my-deployment
  ```

- **Label 추가**:
  ```bash
  kubectl label pod my-pod app=my-app
  ```

- **Annotation 추가**:
  ```bash
  kubectl annotate pod my-pod description="test pod"
  ```
