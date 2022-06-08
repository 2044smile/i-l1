# i-l1

# infra(Kubernetes, EKS, Argo Workflow, Helm)

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
    * --dry-run 을 사용하면 코드를 쉽게 테스트할 수 있지만, Kubernetes가 만든 템플릿을 받아들이는지 확신할 수 없다.
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
  * base64 Encoded **POD**(TOKEN=6rCA64KY64ukCg==) created secrets🔽 - 정리하면 secrets 에서는 Decoded 된 값을 못보는 반면
  * base64 Decoded **Django**(TOKEN=가나다) - Pod 안에서는 Decoded 된 값을 볼 수 있다.

# 일지 (2022-06-08)
 
* 유급휴가 사용 (6월 13일 월요일 수술을 위한 사전 검사 진행)
