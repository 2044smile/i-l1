# i-l1

# infra

Pod 로그 및 status 확인 (done)<br/>
EKS cluster 내에 구성된 pod 내부 로그 및 status 확인 (done)<br/>
<br/>
Argo workflow 수행 (done)<br/>
기구성된 Argo workflow manager 내 특정 workflow 수행 및 status, 로그 확인 (done)<br/>
<br/>
Configmap, Secret 설정 (done)<br/>
Configmap, Secret 설정 / 변경을 통한 pod 내 환경 변수 설정 (done)<br/>
<br/>
Argo 배포 (done)<br/>
기구성된 EKS cluster 내에 Argo workflow manager 배포 (done)<br/>
<br/>
Helm chart 구성 (done)<br/>
EKS내 pod, deployment, service 배포를 위한 helm chart 구성 및 helm chart를 이용한 Kubernetes 서비스 배포 (done)<br/>
<br/>
Argo workflow 구성 및 배포, 수행<br/>
주어진 code를 수행할 수 있는 argo workflow template 구성, 배포 및 Argo workflow manager를 통한 workflow 수행<br/>
<br/>

done

# 추가 과제 발생

1. configmap, secret 실제로 환경변수로 적용됐는지 팟에서 확인 (ex. Echo $환경변수) (done)
2. argo가 PROTO-INFERENCE-NG에 속한 node에만 뜨도록 설정
3. backend 테스트에서 만든 Django project를 helm chart로 띄우기
