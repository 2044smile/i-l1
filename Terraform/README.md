# Terraform

* 현재 공급되는 가장 인기 있는 인프라 자동화 툴 중 하나이며, 이는 단순 구문을 사용하여 여러 클라우드 및 온프레미스 데이터 센터에서 인프라를 프로비저닝하고 구성 변경에 대한 대응으로 인프라를 안전하고 효율적으로 다시 프로비저닝할 수 있기 때문입니다.
* IaC(코드형 인프라)는 버전 관리, 지속적 통합, 지속적 배포 등 Agile 및 DevOps 실무의 주요 구성요소입니다.
* 코드로써의 인프라는 인프라를 이루는 서버, 미들웨어 그리고 서비스 등, 인프라 구성요소들을 코드를 통해 구축하는 것.
* IaC는 코드로써의 장점, 즉 작성용이성, 재사용성, 유지보수 등의 장점을 가진다.

## 기본 구성 요소

* provider: 테라폼으로 생성할 인프라의 종류를 의미, access, secret_key
  * 테라폼과 외부 서비스를 연결해주는 기능을 하는 모듈
* resource: 테라폼으로 실제로 생성할 인프라 자원을 의미, t2.micro, ami
  * provider가 제공해주는 조작 가능한 대상의 최소 단위
  * EX] AWS 가 제공해주는 EC2, Security_Group, Key ...
* state: 테라폼을 통해 생성한 자원의 상태를 의미
* output: 테라폼으로 만든 자원을 변수 형태로 state 에 저장하는 것을 의미
* module: 공통적으로 활용할 수 있는 코드를 문자 그대로 모듈 형태로 정의하는 것을 의미
* remote: 다른 경로의 state 를 참조하는 것을 말한다. output 변수를 불러올 때 주로 사용

## 명령어

* init: 명령어 사용을 위해 각종 설정을 진행
* plan: 작성한 코드가 실제로 어떻게 만들어질지에 대한 예측 결과를 보여준다.
* apply: 코드로 실제 인프라를 생성하는 명령어
* import: 이미 만들어진 자원을 테라폼 state 파일로 옮겨주는 명령어
* state: 하위 명렁어로 mv, push
* destroy: 생성된 자원들 state 파일 모두 삭제하는 명렁어

## 프로세스

1. init
   1. 내부적으로 provider 와 state, module 설정
2. plan
   1. 예측 결과를 보여주는 명령어
   2. 기본적으로 plan 에 문제가 없어야 apply 에 문제가 없을 확률이 높다
3. apply
   1. 실제로 작성한 코드로 인프라를 생성하는 명령어
   2. 실제 인프라에 영향을 끼치는 명령어이므로 주의깊게 실행을 해야한다.

## 테스트

```markdown
# main.tf
resource "fakewebservices_server" "servers" {
  count = 2 --> 1  # successed

  name = "Server ${count.index + 1}"
  type = "t2.micro"
  vpc  = fakewebservices_vpc.primary_vpc.name
}
```


### Refernece

* https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code