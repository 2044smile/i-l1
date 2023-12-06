# i-l1

# infra(Kubernetes, EKS, Argo Workflow, Helm[패키지 관리자])

- low - i-l1 - Pod 로그 및 status 확인, EKS cluster 내에 구성된 pod 내부 로그 및 status 확인 (done)<br/>
- low - i-l2 - Argo workflow 수행, 기구성된 Argo workflow manager 내 특정 workflow 수행 및 status, 로그 확인 (done)<br/>
- mid - i-m1 - Configmap, Secret 설정, Configmap, Secret 설정 / 변경을 통한 pod 내 환경 변수 설정 (done)<br/>
- mid - i-m2 - Argo 배포, 기구성된 EKS cluster 내에 Argo workflow manager 배포 (done)<br/>
- high - i-h1 - Helm chart 구성, EKS내 pod, deployment, service 배포를 위한 helm chart 구성 및 helm chart를 이용한 Kubernetes 서비스 배포 (done)<br/>
- high - i-h2 - Argo workflow 구성 및 배포, 수행, 주어진 code를 수행할 수 있는 argo workflow template 구성, 배포 및 Argo workflow manager를 통한 workflow 수행(done)<br/>

# 추가 과제 발생

* configmap, secret 실제로 환경변수로 적용됐는지 팟에서 확인 (ex. Echo $환경변수) (done)
* argo가 PROTO-INFERENCE-NG에 속한 node에만 뜨도록 설정 (done)
* backend 테스트에서 만든 Django project를 helm chart로 띄우기

# 일지 (2022-06-03)

* docker Repository <none>, Tag <none> | docker tag 'docker image ID 4자리' REPOSITORY:TAG
* Docker -> ECR -> Helm 

# 일지 (2022-06-07)

* Docker -> ECR -> **Helm**
  * TMI: 쿠버네티스에 서비스를 배포하기 위해 사용하는 대표적인 방법중에 하나가 바로 Helm chart 이다.
  * helm install --debug <name> .charts --namespace <namespace> EX) node-group=A, 
    * '{"spec": {"template": {"spec": {"nodeSelector": {"Node-Selectors": "A"}}}}}'
    * '{"spec": {"template": {"Node-Selector": "node-group=A"}}}'
    * --dry-run 을 사용하면 코드를 쉽게 테스트할 수 있지만, Kubernetes가 만든 템플릿을 받아들이는지 확신할 수 없다
  * helm _helpers.tpl 템플릿 정의 할 때 참고하는 값
* Kubernetes patch deployments/<deployment> -p '{"A":"a"}' 와 같이 값을 설정할 수 있다.
* Kubernetes(Docker) Pull Error 시 image: 의 name을 체크해봐야한다.
* "A-secrets" not found 의 경우 envFrom, secretRef를 사용하여 시크릿이 설정되어 있다는 뜻이다.
  * kubectl create secret generic A-secrets 로 생성해주면 된다.
* AWS profile 설정
  * aws configure --profile $PROFILE_NAME cli 에서 위 명령어 실행 후 각자 access key 등 입력
* EKS 설정
  * aws eks --region $REGION update-kubeconfig --name $CLUSTER_NAME --profile $PROFILE_NAME
* EKS 설정 확인
  * kubectl config current-context ex) arn:aws:eks:ap-northeast-2:************:cluster/A
* 최종 확인
  * kubectl config set-context --current --namespace=A
  * kubectl get pod
* Kubernetes secrets 
  * base64 Encoded **POD**(TOKEN=6rCA64KY64ukCg==) created secrets - 정리하면 secrets 에서는 Decoded 된 값을 못보는 반면
  * base64 Decoded **Django**(TOKEN=가나다) - Pod 안에서는 Decoded 된 값을 볼 수 있다.

# Helm

### environments 폴더는 모든 환경 관련 파일을 저장하는 곳이다. [configMap, Secret, values]
* values
  * helm upgrade dodo-helm-tutorial -f environments/values.dev.yaml .
  * helm upgrade dodo-helm-tutorial -f environments/values.prod.yaml .
* Secret
  * Secret 파일은 git 에 체크인[add .helmignore되지 않으며 CI/CD 파이프라인에서만 사용할 수 있다.
  * Secret 파일 내의 각 키 값은 **base64** 로 인코딩되어야 한다.

### 조건문과 반복문

* if
  ```yaml
  {{ if or (eq .Values.env "prod") }}  # dev(-f values.dev.yaml) 에서는 아래 SENTRY__DSN 의 값을 볼 수 없다
  SENTRY__DSN: https://sentry.io/
  {{ end }}
  ```

* range는 list와 tutple 외에도 map, dict를 반복하는데 사용할 수 있습니다.
  ```yaml
  {{ range $index, $origin := .Values.web.crossOrigins }}
  WEB__CROSS_ORIGIN__{{ $index }}: {{ $origin }}
  {{ end }}
  ```

# Chart hooks

* 서비스가 처음 배포될 때 돌아야 하는 **데이터베이스 마이그레이션 Job** 이었습니다.
* Helm 에서는 hooks 를 이용하여 릴리즈 생명주기에 개입 할 수 있습니다.
* install, delete, upgrade, rollback 각각 실행 전후로 hook 지정이 가능합니다.

```yaml
# templates/post-install.job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration-job
  annotations:
    "helm.sh/hook": post-install  # 어떤 시점에 hook을 동작시킬 지 지정할 수 있으며 ',' 를 통해 여러 시점을 지정 할 수 있습니다.
    "helm.sh/hook-weight": "5"  # 실행 순서 결정을 위하여 정수를 통해 중요도를 지정 할 수 있으며 문자열로 정의해야 합니다.
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded  # hook을 언제 삭제할지 설정할 수 있으며 지정하지 않을 경우 다음 hook 이 생성될 때 사라집니다.
spec:
```

# plugin

* helm diff 플러그인을 사용하면 CI 단계에서 배포 전후의 k8s 리소스에 어떤 변화가 일어나는지 미리 체크하는 목적으로 유용하게 활용

# Kubernetes

* kubectl get rs: replicas 정보 확인
* kubectl rollout undo deploy nginx-deployment: 배포 롤백하기

## 배포[Deployment] 전략

* 기본적으로 RollingUpdate 방식으로 배포를 하지만, Blue/Green 는 Istio, Nginx 또는 Argo 와 같은 도구를 사용하여 Blue/Green 배포를 많이 진행한다.

1. Recreate Deployment
  * 현재 실행 중인 pod 인스턴스를 종료하고 새 버전으로 동시에 새로 실행한다. 
  * 애플리케이션 중단을 막을 수 없으며, 복구(롤백) 프로세스를 돌리는데에도 시간지연이 발생한다.
  * 간단하고 빠르고 저렴하다는 장점이 있다. 하지만 가장 위험한 배포전략으로 모범 사례에 속하지 않는다. 
  * 서비스가 비즈니스, 미션, 수익에 중요하지 않는 경우, 근무 시간외에 배포해도 괜찮은 경우, 개발이나 스테이징 환경 등에 주로 사용한다.

2. **Rolling Update Deployment [무중단 배포]**
  * 롤링 업데이트는 실행 중인 애플리케이션 인스턴스를 순차적으로 새 릴리즈로 업데이트하는 배포 전략이다.
  * 롤릴 업데이트는 롤백이 비교적 간단하며 Recreate Deployment 보다 더 안전하다는 장점이 있다. 
  * 하지만 릴리즈 업데이트가 완료되기 전까지, 서로 다른 버전의 애플리케이션이 실행될 수 있으므로 여기에서 발생할 수 있는 문제점을 해결 할 수 있어야 한다.
    * LET ME] 즉, 이전 버전과 새로운 버전이 공존하는 시간이 발생하지만, 안정적으로 이루어지기 위한 방법 중 하나입니다.
    * LET ME] 롤백을 시도할때도 복구 시간이 소요된다.
  * 업데이트 할 때, 각 애플리케이션이 성공적으로 배포됐는지를 확인하기 때문에 배포에 시간이 걸릴 수 있다.

3. Blue/Green Deployment
  * 블루/그린 배포는 서로 다른 버전의 애플리케이션(혹은 서비스)를 블루(일명 스테이징)와 그린(일명 프로덕션) 두개의 환경을 활용하는 배포 전략이다. 기능 테스트, 품질관리 등의 활동은 블루 환경에서 수행된다. 새로운 변경이 수용되고 테스트가 끝나면, 트래픽을 블루로 라우팅한다.
  * 블루/그린 배포는 간단하고 빠르며 이해하기가 쉽고 구현이 쉽다는 장점이 있다. 롤백도 간단하다. 문제가 발생하며 트래픽을 이전 환경으로 되돌리기만 하면 된다. 다른 배포 전략에 비해서 매우 안전하다.
  * 블루/그린의 단점은 비용에 있다. 마이크로 서비스 환경에서 복제된 두 개의 환경을 구축하는데에는 많은 시간과 비용이 들어갈 수 있다. 품질관리, 승인테스트, 회귀테스트를 수행하긴 하지만 모든 사용자 트래픽을 한 번에 전환하면 위험이 발생 할 수 있다. 따라서 블루/그린 환경을 제대로 구성하기 위해서는 잦은 배포 전략을 유지할 필요가 있다.

4. Canary Deployment
  * Canary 배포는 서비스를 일부 사용자에게 점진적으로 릴리즈 하는 배포 전략이다. 대상 환경의 인프라는 작은 단계(25%, 50%, 75%, 100%)로 업데이트 한다. Canary 배포는 다른 배포 전략에 비해서 가장 안전하다.
  * Canary 배포를 통해 조직은 실제 사용자를 대상으로 현재 버전과 업데이트 버전의 작동 경험을 함께 비교 할 수 있다. 블루/그린 배포처럼 복제를 만들 필요가 없기 때문에 비용 역시 저렴하다. 롤백역시 빠르고 안전하다.
  * Cnary 배포는 프로덕션 환경에서의 테스트에 많은 기술적 노하우가 필요하다는 단점을 가진다. Canary 스크립트는 복잡 하며, 테스트에 많은 시간이 걸릴 수 있다. 테스트 전략, 모니터링 등에 추가 연구가 필요 할 수 있다.

### Reference 

* https://spoqa.github.io/2020/03/30/k8s-with-helm-chart.html
* https://www.joinc.co.kr/w/kubernetes_minikube_index