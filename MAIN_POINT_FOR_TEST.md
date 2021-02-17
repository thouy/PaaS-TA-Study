# BOSH Concept
## 1. BOSH Director (혹은 BOSH VM)
CLI를 통해 명령을 받으면 IaaS 환경에 VM을 기반으로 작업을 수행합니다.
## 2. BOSH Agent
Bosh가 설치한 VM마다 배치된 Agent. 각 VM 시스템의 오류를 감지하여 경고하거나 자동으로 문제를 복구하는 역할을 합니다.
## 3. BOSH Release
VM에 설치할 소프트웨어 패키지, 설정 템플릿, 프로세스 시작/종료 스크립트를 포함합니다. (주로 소프트웨어 패키지)
## 4. BOSH Stemcell
VM에 설치할 OS 이미지 및 BOSH Agent를 포함합니다. (주로 OS 이미지)
## 5. BOSH 배포 Manifest 파일
VM 배포에 필요한 정보를 정의한 YAML 포맷의 파일. (적용할 Stemcell 및 Release 정보, VM의 용량, 네트워크 구성정보 등..)

# BOSH CLI command

## 1. BOSH VM 배포(설치) command : create-env
~~~sh
$ bosh create-env bosh.yml \
	--state=warden/state.json \
	--vars-store warden/creds.yml \
	-o virtualbox/cpi.yml \
	-o virtualbox/outbound-network.yml \
	-o bosh-lite.yml \
	-o bosh-lite-runc.yml \
	-o uaa.yml \
	-o credhub.yml \
	-o jumpbox-user.yml \
	-v inception_os_user_name='ubuntu' \
	-v internal_cidr='10.0.1.0/24' \
	-v internal_gw='10.0.1.1' \
	-v internal_ip='10.0.1.6' \
	-v network_name='vboxnet0' \
	-v outbound_network_name='NatNetwork'
~~~
## 2. BOSH VM에 별칭 및 인증정보 설정 : alias-env
~~~sh
$ bosh alias-env micro-bosh \
	-e 10.0.1.6
	--ca-cert <(bosh int warden/creds.yml --path /director_ssl/ca)
~~~
## 3. BOSH VM에 로그인하기
~~~sh
#!/usr/bin

export BOSH_CA_CERT=$(bosh int ./warden/creds.yml --path /director_ssl/ca)
export BOSH_CLIENT=admin
export BOSH_CLIENT_SECRET=$(bosh int ./warden/creds.yml --path /admin_password)
~~~
## 3. BOSH VM 환경 및 상태 조회 : env (--details 속성을 주면 인증서 정보까지 조회 가능)
~~~sh
$ bosh -e micro-bosh env
~~~
## 4. BOSH VM에 SSH로 접속하기 : jumpbox 활용
~~~sh
$ bosh int creds.yml --path /jumpbox_ssh/private_key > jumpbox.key
$ chmod 600 jumpbox.key
$ ssh jumpbox@10.0.1.6 -i jumpbox.key
~~~
## 5. BOSH VM에 cloud-config 적용하기 : update-cloud-config
~~~sh
$ bosh -e micro-bosh update-cloud-config cloud-config.yml
~~~
## 6. BOSH VM에 runtime-config 적용하기 : update-runtime-config
~~~sh
$ bosh -e micro-bosh update-runtime-config runtime-config.yml
~~~
## 7. BOSH VM에 steamcell 적용하기 : upload-stemcell
~~~sh
$ bosh -e micro-bosh upload-stemcell stemcell.tgz
~~~
## 8. BOSH VM에 적용된 config 조회 : configs
~~~sh
$ bosh -e micro-bosh configs
~~~
## 9. BOSH VM에 적용된 stemcell 조회 : stemcells
~~~sh
$ bosh -e micro-bosh stemcells
~~~

# Cloud Foundry CLI command
## 1. cf에 로그인 하기
~~~sh
$ cf login --skip-ssl-validation
~~~
## 2. user, org, space 생성하기 : create-user, create-org, create-space
~~~sh
$ cf create-user USERNAME PASSWORD
$ cf create-org ORGNAME
$ cf create-space -o ORGNAME SPACENAME
~~~
## 3. org, space에 role 설정하기 : set-org-role, set-space-role
~~~sh
$ cf set-org-role USERNAME ORGNAME ROLE  # ROLE : OrgManager, BillingManager, OrgAuditor
$ cf set-space-role USERNAME ORGNAME SPACENAME ROLE # ROLE : SpaceManager, SpaceDeveloper, SpaceAuditor
~~~
## 4. space를 org에 매핑하기 : target
~~~sh
$ cf target -o ORGNAME -s SPACENAME
~~~
## 5. 앱 푸시 : push (manifest.yml 파일 필수, -b 속성으로 커스텀 빌드팩 적용 가능)
~~~sh
$ cf push
~~~
## 6. 해당 space에 등록된 앱 목록 조회 : apps
~~~sh
$ cf apps
~~~
## 7. 앱 현재 상태 및 health 정보 확인 : app
~~~sh
$ cf app APPNAME
~~~
## 8. 앱의 환경변수, 바인드 된 서비스 정보 확인 : env
~~~sh
$ cf env APPNAME
~~~
## 9. 앱의 로그 보기 : logs
~~~sh
$ cf logs APPNAME    # 앱의 tail 로그
$ cf logs APPNAME --recent   # 앱의 최근 로그
~~~
## 10. 해당 space에 등록된 서비스 목록 조회 : services
~~~sh
$ cf services
~~~
## 11. 서비스 인스턴스 생성하기 : create-service
~~~sh
$ cf create-service SERVICENAME PLAN SERVICE_INSTANCE_NAME
~~~
## 12. 앱과 서비스 인스턴스 바인딩하기 : bind-service
~~~sh
$ cf bind-service APPNAME SERVICE_INSTANCE_NAME
~~~
## 13. 바인딩 후에 앱에 반영하기 : restage
~~~sh
$ cf restage APPNAME
~~~
## 14. 서비스 브로커 목록 조회 : service-brokers
~~~sh
$ cf service-brokers
~~~
## 15. 서비스 브로커 등록하기 : create-service-broker
~~~sh
$ cf create-service-broker SERVICE_NAME USER_ID USER_PW SERVICE_URL   # 등록되면 marketplace에서 조회됨
~~~

# Service Broker
| Service Broker API | Routes (API)                                   | Method | 설명                                                       |
| ------------------ | ---------------------------------------------- | ------ | ---------------------------------------------------------- |
| catalog            | /v2/catalog                                    | GET    | 서비스 및 서비스 Plan 정보 조회                            |
| provision          | /v2/service_instances/:id                      | PUT    | 서비스를 위한 인스턴스 생성                                |
| deprovision        | /v2/service_instances/:id                      | DELETE | 서비스 인스턴스 삭제                                       |
| updateprovision    | /v2/service_instances/:id                      | PATCH  | 서비스 인스턴스 Plan을 수정                                |
| bind               | /v2/service_instances/:id/service_bindings/:id | PUT    | 서비스 사용에 관련된 사용자 생성 및 권한 등 설정 정보 생성 |
| unbind             | /v2/service_instances/:id/service_bindings/:id | DELETE | 서비스 사용 설정 정보 삭제                                 |

# Buildpack
* 애플리케이션 구동이 필요한 환경(런타임, 프레임워크 등)을 조립하고 드롭릿을 구성하는 스크립트의 모음
* 검출 : 배포된 애플리케이션의 런타임 환경 구성 방법을 빌드팩이 아는지 여부를 확인하는 기능
* 컴파일 : 실질적으로 드롭릿을 빌드하는 빌드팩의 핵심기능
* 릴리즈 : 애플리케이션 실행방법에 대한 정보를 플랫폼에 응답해주는 기능
* 패키지 : 빌드팩을 하나의 압축파일로 만드는 기능 혹은 압축파일 자체
* cf로 앱을 푸시할 때 -b 옵션으로 빌드팩을 지정할 수 있음. 원격지에 있는 빌드팩도 URL 값으로 지정 가능 (ex. Git URL 등)