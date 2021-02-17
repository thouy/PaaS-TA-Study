# Cloud Foundry
* 오픈소스 기반 멀티 클라우드 애플리케이션 플랫폼
* Clound Foundry 재단에 의해 관리되고 있습니다.

# Bosh
* IaaS 환경에 가상머신(VM) 들을 설치하고 관리하는 도구
* Cloud Foundry 플랫폼 기반에서 애플리케이션을 다수의 VM으로 배포할 수 있는 배포 도구
* 다양한 벤더의 laaS 인프라를 지원합니다. (Amazon EC2, OpenStack, vSphere 등)
* 구성요소
	1. BOSH CLI : BOSH Director와 상호작용하는 Command Line Interface
	2. BOSH Director (혹은 BOSH VM) : CLI를 통해 명령을 받으면 IaaS 환경에 VM을 기반으로 작업을 수행합니다.
	3. BOSH Agent : Bosh가 설치한 VM마다 배치된 Agent. 각 VM 시스템의 오류를 감지하여 경고하거나 자동으로 문제를 복구하는 역할을 합니다.
	4. BOSH Release : VM에 설치할 소프트웨어 패키지, 설정 템플릿, 프로세스 시작/종료 스크립트를 포함
	5. BOSH Stemcell : VM에 설치할 OS 이미지 및 BOSH Agent를 포함
	6. BOSH 배포 Manifest 파일 : VM 배포에 필요한 정보를 정의한 YAML 포맷의 파일. (적용할 Stemcell 및 Release 정보, VM의 용량, 네트워크 구성정보 등..)

# 실습환경 구성
* 실습환경에 대한 설명
	1. 본래는 IaaS 기반에 Bosh VM을 포함한 다수의 VM이 있고, Bosh-cli를 통해 Bosh VM(Director) 으로 명령을 실행하면서 운영하는 구조 입니다.
	2. 실습환경에서는 IaaS를 통해 수 개의 VM을 운영하는 것은 비용적인 부담이 따르기 때문에 로컬 환경에서의 실습을 위한 Bosh-lite를 활용합니다. (Bosh-lite는 실습환경을 위한 프로그램)
	3. Bosh-lite를 이용하여 하나의 서버에 IaaS 환경까지 구축할 수 있습니다.
	4. Bosh-cli와 Virtual Box를 설치한 후, Bosh-cli를 통해 Bosh-lite를 이용하여 Bosh VM을 생성합니다. 
	5. Bosh-lite는 Bosh VM과 같은 역할을 수행하며, 컨테이너의 형태로 PaaS-TA를 배포합니다.
* 사전 준비 사항
	1. Ubuntu Desktop : SSH 접속이 가능해야 하므로 openssh-server가 설치되어 있어야 합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ sudo apt install openssh-server
	~~~
	2. 윈도우 환경 사용자라면 Ubuntu Desktop 구성을 위해 VMWare Workstation Player 설치가 필요합니다.
	3. curl 패키지를 설치합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ sudo apt install curl
	~~~
	4. 홈 디렉토리 밑에 workspace 디렉토리를 생성합니다.
* Dependency 설치
	1. 필요한 Dependency를 설치하는 아래 명령을 실행합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt1-dev libxml2-dev libssl-dev libreadline7 libreadline-dev libyaml-dev libsqlite3-dev sqlite3 
	~~~
* Paas-TA 설치파일 다운로드
	1. curl을 이용하여 workspace 밑에 paasta-5.0.zip 파일을 다운로드 받습니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ curl -Lo paasta-5.0.zip https://nextcloud.paas-ta.org/index.php/s/Qi2zGPnGNEjb4Ax/download
	~~~
	2. paasta-5.0.zip 파일의 압축을 해제합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ unzip paasta-5.0.zip
	~~~
	3. 디렉토리명을 paasta-5.0으로 변경합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ mv bosh-lite paasta-5.0
	~~~
* Bosh CLI 설치
	1. Bosh cli 다운로드
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ curl -Lo ./bosh https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-6.1.0-linux-amd64
	~~~
	2. bosh 디렉토리를 실행가능한 권한으로 변경합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ chmod +x ./bosh
	~~~
	3. bosh 명령을 바로 실행할 수 있도록 /usr/local/bin 디렉토리로 이동시킨다. 
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ sudo mv ./bosh /usr/local/bin/bosh
	~~~
	4. 완료되었으면 bosh가 잘 동작하는지 버전체크 명령 실행 (bosh -v)
* Bosh 설치 (Bosh Director 설치)
	1. bosh create-env 명령어로 설치합니다.
	2. 설치하게 되면 Virtual Box에 VM을 생성하여 해당 VM을 Bosh Director로 설정하게 됩니다.
	3. Paas-TA 설치파일 다운로드 섹션에서 세팅된 paasta-5.0 디렉토리의 deployment/bosh-deployment 디렉토리에 IaaS 벤더별 bosh 설치 쉘 스크립트가 있습니다. (deploy-벤더명.sh 파일)
	4. bosh-depolyment 디렉토리 내의 sh 파일들의 권한을 변경합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ chmod 755 *.sh
	~~~
	5. 실습환경에서의 설치를 위해 deploy-bosh-lite.sh 명령을 실행시킵니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ ./deploy-bosh-lite.sh
	~~~
	6. deploy-bosh-list.sh 내용
	~~~sh
	#!/bin/bash
	bosh create-env bosh.yml \
        --state=warden/state.json \  # bosh 설치 시 생성되는 파일로 절대 삭제하면 안됨 (Backup 권장)
        --vars-store warden/creds.yml \   # bosh 내부 인증서 파일 (Backup 권장)
        -o virtualbox/cpi.yml \      # Virtual Box CPI를 적용하겠음
        -o virtualbox/outbound-network.yml \
        -o bosh-lite.yml \
        -o bosh-lite-runc.yml \
        -o uaa.yml \
        -o credhub.yml \     # 인증정보 저장소 설정. 추후 PaaS-TA 설치 시 인증정보가 이 파일에 저장됨
        -o jumpbox-user.yml \
        -v inception_os_user_name='ubuntu' \  # Inception 서버의 유저명
        -v director_name='micro-bosh' \
        -v internal_cidr='10.0.1.0/24' \   # Internal(사설망) IP 대역
        -v internal_gw='10.0.1.1' \     # Internal Gateway
        -v internal_ip='10.0.1.6' \     # Internal IP Address
        -v network_name='vboxnet0' \
        -v outbound_network_name='NatNetwork'
	~~~
	
* Virtual Box 설치
	1. IaaS 환경의 대안으로 활용할 Virtual Box를 설치합니다.
	1. 아래의 명령을 이용하여 Virtual Box 설치를 위한 repository를 추가합니다.
	~~~sh
ubuntu@ubuntu-virtual-machine:~/workspace$ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
ubuntu@ubuntu-virtual-machine:~/workspace$ wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
ubuntu@ubuntu-virtual-machine:~/workspace$ sudo add-apt-repository "deb http://download.virtualbox.org/virtualbox/debian bionic contrib" 
	~~~
	2. repository 추가후엔 반드시 apt update 명령으로 추가한 repository 정보를 반영해줍니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ sudo apt update
	~~~
	3. Virtual Box 6.0을 설치한다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ sudo apt install virtualbox-6.0
	~~~
	4. 설치가 잘 되었는지 아래의 명령으로 확인합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ VBoxManage --version
	~~~

# BOSH VM 설정 및 접속하기
* BOSH VM에 로그인 하기
	1. 설치한 BOSH VM에 별칭 및 인증정보를 미리 설정합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ bosh alias-env micro-bosh -e 10.0.1.6 --ca-cert <(bosh int warden/creds.yml --path /director_ssl/ca) 
	~~~
	2. BOSH VM에 로그인을 위한 login.sh 파일을 작성합니다.
	~~~sh
	#!/usr/bin
	
	export BOSH_CA_CERT=$(bosh int ./warden/creds.yml --path /director_ssl/ca)
	export BOSH_CLIENT=admin
	export BOSH_CLIENT_SECRET=$(bosh int ./warden/creds.yml --path /admin_password) 
	~~~
	3. 작성한 login.sh 파일을 적용합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ source login.sh
	~~~
	4. 로그인이 잘 적용되었는지 아래의 명령으로 확인합니다. User 항목에 admin이라고 입력되어 있다면 로그인이 잘 적용된 상태입니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ bosh -e micro-bosh env
	
	Using environment '10.0.1.6' as client 'admin'

	Name               micro-bosh  
	UUID               b6af9ed7-0542-4751-853d-6edeba344f43  
	Version            270.2.0 (00000000)  
	Director Stemcell  ubuntu-xenial/315.64  
	CPI                warden_cpi  
	Features           compiled_package_cache: disabled  
	                   config_server: enabled  
	                   local_dns: enabled  
	                   power_dns: disabled  
	                   snapshots: disabled  
	User               admin  

	Succeeded
	~~~
* Bosh VM에 SSH로 접속하기 위한 준비 (jumpbox를 이용하여 key 생성)
	1. 아래의 명령으로 jumpbox.key 파일을 생성합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ bosh int warden/creds.yml --path /jumpbox_ssh/private_key > jumpbox.key 
	~~~
	2. jumpbox.key 파일의 권한을 600으로 변경합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ chmod 600 jumpbox.key
	~~~
	3. SSH 명령으로 Bosh VM에 접속합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ ssh jumpbox@10.0.1.6 -i jumpbox.key
	
	Unauthorized use is strictly prohibited. All access and activity
is subject to logging and monitoring.
	Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-54-generic x86_64)

	 * Documentation:  https://help.ubuntu.com
	 * Management:     https://landscape.canonical.com
	 * Support:        https://ubuntu.com/advantage
	Last login: Mon Feb 15 09:07:58 UTC 2021 from 10.0.1.1 on pts/0
	Last login: Mon Feb 15 15:20:01 2021 from 10.0.1.1
	bosh/0:~$
	~~~
	4. SSH 접속에 성공하면 root 권한을 획득하고 나면 명령어를 통해 Bosh VM을 관리할 수 있습니다. 아래는 root 권한 획득 후, Bosh VM이 배포한 프로세스를 조회하는 모습입니다.
	~~~sh
	bosh/0:~$ sudo su
	bosh/0:/home/jumpbox# monit summary
	The Monit daemon 5.2.5 uptime: 6h 25m 

	Process 'nats'                      running
	Process 'postgres'                  running
	Process 'blobstore_nginx'           running
	Process 'director'                  running
	Process 'worker_1'                  running
	Process 'worker_2'                  running
	Process 'worker_3'                  running
	Process 'worker_4'                  running
	Process 'director_scheduler'        running
	Process 'director_sync_dns'         running
	Process 'director_nginx'            running
	Process 'health_monitor'            running
	Process 'garden'                    running
	Process 'warden_cpi'                running
	Process 'uaa'                       running
	Process 'credhub'                   running
	System 'system_localhost'           running
	bosh/0:/home/jumpbox#
	~~~
	5. SSH 접속을 끊을 때는 exit 명령을 실행해줍니다.
* Bosh VM에 credhub 설치 및 설정하기
	1. wget 명령으로 credhub cli를 다운로드 합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ wget https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.0.0/credhub-linux-2.0.0.tgz
	~~~
	2. 다운로드한 파일의 압축을 해제하고 실행 가능한 권한으로 변경해줍니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ tar -xvf credhub-linux-2.0.0.tgz
	ubuntu@ubuntu-virtual-machine:~$ chmod +x credhub 
	~~~
	3. credhub 명령을 바로 실행할 수 있도록 /usr/local/bin 디렉토리로 이동시킵니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ sudo mv credhub /usr/local/bin/credhub 
	~~~
	4. credhub cli가 잘 설치되었는지 아래의 명령으로 확인합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ credhub --version
	~~~
	5. credhub 로그인 쉘 파일을 만들기 위해 bosh-deployment 디렉토리에 아래의 내용으로 credhub_login.sh 파일을 생성합니다.
	~~~sh
	#!/usr/bin

	export CREDHUB_CLIENT=credhub-admin 
	export CREDHUB_SECRET=$(bosh int --path /credhub_admin_client_secret warden/creds.yml) 
	export CREDHUB_CA_CERT=$(bosh int --path /credhub_tls/ca warden/creds.yml) 
	~~~
	6. 아래의 명령으로 credhub_login.sh을 적용시켜줍니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ source credhub_login.sh
	~~~
	7. 아래의 명령으로 credhub에 로그인할 수 있습니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ credhub login -s https://10.0.1.6:8844 --skip-tls-validation 
	Warning: The targeted TLS certificate has not been verified for this connection.
	
	Warning: The --skip-tls-validation flag is deprecated. Please use --ca-cert instead.
	Setting the target url: https://10.0.1.6:8844
	Login Successful
	~~~
	8. credhub 로그인 완료 후, credhub --version 명령을 실행하면 credhub 서버의 버전도 조회가 가능합니다.
	9. credhub find 명령으로 credential 정보를 조회할 수 있습니다. Bosh VM에 의해 새로운 VM이 생성되면 해당 VM의 인증정보가 credhub로 저장됩니다.
* Bosh VM을 설치(배포)한 후에는 Bosh VM을 임의로 Power Off 하거나 중지시킬 수 없습니다. 반드시 아래의 명령으로 Bosh VM의 상태를 저장한 후 정지시켜 줍니다. 아래 명령은 bosh-deployment 디렉토리에서 실행시켜줍니다.
~~~sh
ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ vboxmanage controlvm $(bosh int warden/state.json --path /current_vm_cid) savestate

~~~
* 정지시킨 Bosh VM을 다시 Run 하려면 아래의 명령을 실행시켜 줍니다.
~~~sh
ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ vboxmanage startvm $(bosh int warden/state.json --path /current_vm_cid) --type headless
~~~

# PaaS-TA 구축하기
* PaaS-TA를 설치(배포)하기 위해서 먼저 Bosh에 로그인 합니다.
~~~sh
ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ source login.sh
~~~
* cloud-config 설정하기
	1. PaaS-TA 배포는 cloud-config 설정에 따라 배포가 진행되기 때문에, Bosh VM에 먼저 업로드 되어야 PaaS-TA 배포가 가능합니다.
	2. cloud-config는 IaaS 환경에 따라 적절한 설정값을 사용해야 합니다.
	3. 실습환경 기준으로 bosh-lite-cloud-config.yml 파일을 Bosh VM에 적용합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/cloud-config$ bosh -e micro-bosh update-cloud-config bosh-lite-cloud-config.yml
	~~~
	4, cloud-config 적용이 완료되면 bosh -e micro-bosh cloud-config 명령으로 적용된 cloud-config 내용을 확인할 수 있습니다.
* runtime-config 설정하기
	1. bosh-deployment 디렉토리에 update-runtime-config.sh을 실행시킵니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ ./update-runtime-config.sh
	~~~
	2. runtime-config 적용이 완료되면 bosh -e micro-bosh runtime-config 명령으로 적용된 runtime-config 내용을 확인할 수 있습니다.
* 2가지의 config가 잘 적용되었는지 확인하려면 아래의 명령을 입력하여 확인할 수 있습니다.
~~~sh
ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/bosh-deployment$ bosh -e micro-bosh configs

Using environment '10.0.1.6' as client 'admin'

ID  Type     Name     Team  Created At  
1*  cloud    default  -     2021-02-15 09:23:23 UTC  
2*  runtime  default  -     2021-02-15 09:27:29 UTC  

(*) Currently active
Only showing active configs. To see older versions use the --recent=10 option.

2 configs

Succeeded
~~~
* stemcell 업로드하기
	1. paasta-5.0/stemcell/passta 디렉토리에 있는 stemcell 파일을 Bosh VM으로 업로드 합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/stemcell/paasta$ bosh -e micro-bosh upload-stemcell bosh-stemcell-315.64-warden-boshlite-ubuntu-xenial-go_agent.tgz
	~~~
	2. 업로드가 완료되면 stemcell 정보를 확인합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/stemcell/paasta$ bosh -e micro-bosh stemcells
	
	Using environment '10.0.1.6' as client 'admin'

	Name                                         Version  OS             CPI  CID  
	bosh-warden-boshlite-ubuntu-xenial-go_agent  315.64*  ubuntu-xenial  -    9edddb03-f8eb-4c1e-66fb-d2e96a0c2bc3  

	(*) Currently deployed

	1 stemcells

	Succeeded
	~~~
* PaaS-TA 배포하기 (시간이 다소 소요됩니다.)
	1. paasta-deployment 디렉토리로 이동하여 deploy-bosh-lite.sh 파일을 실행합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/paasta-5.0/deployment/paasta-deployment$ ./deploy-bosh-lite.sh
	~~~~
* ip route 설정하기 (Bosh lite를 이용한 경우에만 해당합니다.)
	1. PaaS-TA는 10.244.0.0 대역에 배포되어 있고, Bosh VM은 10.0.1.6에 배포가 되어 있어 정상 동작을 위해 ip route 설정을 추가합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ sudo ip route add 10.244.0.0/16 via 10.0.1.6
	~~~
	2. inception 서버 재부팅 후에 명령이 동작하지 않는다면 ip route를 확인한 후 다시 추가하도록 합니다.
* bosh 명령어 활용하기
	1. Paas-TA 배포할 때 새로 생성된 VM 리스트 조회하기 (Bosh VM은 제외한 나머지 목록이 조회됨) : bosh -e micro-bosh -d paasta vms --vitals
	2. PaaS-TA에서 인스턴스로 생성한 VM 리스트 조회하기 : bosh -e micro-bosh instances
	3. Bosh VM의 환경 및 로그인 정보, 인증서 정보까지 조회하기 : bosh -e micro-bosh env --details
	4. 업로드 한 stemcell 목록 조회하기 : bosh -e micro-bosh stemcells
	5. Bosh VM에 올려진 release 목록 조회하기 : bosh -e micro-bosh releases
	6. 최근에 사용했던 task 정보 조회하기 : bosh -e micro-bosh tasks --recent
	7. lock이 걸린 task 정보 조회하기 : bosh -e micro-bosh locks
	8. 동작중인 123번 task 취소하기 : bosh -e micro-bosh cancel-task 123
	9. Paas-TA VM에 SSH로 접속하기 : bosh -e micro-bosh -d paasta ssh 인스턴스명/VM의 ID  (VM의 ID는 다중화 등으로 같은 이름의 인스턴스가 여러개 일 때 명시합니다)
	10. 각 PaaS-TA VM의 로그 디렉토리 경로 : /var/vcap/sys/log 디렉토리

# Cloud Foundary를 이용하여 PaaS-TA VM에 앱 배포하기
* PaaS-TA를 배포하면 Cloud Foundary 프로그램이 자동으로 설치되는데 이 프로그램을 활용하여 PaaS-TA에 앱을 배포할 수 있습니다.
* cf cli 설치하기
	1. 아래의 명령으로 repository를 세팅한 후 cf-cli를 설치합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
	ubuntu@ubuntu-virtual-machine:~$ echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
	ubuntu@ubuntu-virtual-machine:~$ sudo apt update
	ubuntu@ubuntu-virtual-machine:~$ sudo apt install cf-cli
	~~~
	2. cf login 명령을 이용하여 PaaS-TA에 접속합니다. API endpoint는 PaaS-TA 설치시에 사용했던 deploy-bosh-lite.yml 파일에서 도메인 항목에 입력했던 값을 넣어줍니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ cf login --skip-ssl-validation
	API endpoint: https://api.10.244.0.34.xip.io

	Email: admin

	Password: 
	Authenticating...
	OK
	~~~
	3. cf에 앱을 배포할 사용자, org, space를 생성하고 role 및 target을 설정합니다. (--help 속성을 명령 뒤에 추가하면 명령어 사용법을 확인할 수 있습니다.)
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ cf create-user edu-user user
	ubuntu@ubuntu-virtual-machine:~$ cf create-org edu-org
	ubuntu@ubuntu-virtual-machine:~$ cf create-space -o edu-org edu-space
	ubuntu@ubuntu-virtual-machine:~$ cf set-org-role edu-user edu-org OrgManager
	ubuntu@ubuntu-virtual-machine:~$ cf set-space-role edu-user edu-org edu-space SpaceDeveloper
	ubuntu@ubuntu-virtual-machine:~$ cf target -o edu-org -s edu-space
	~~~
	4. 샘플 자바 어플리케이션을 배포하기 위해 Java 8을 설치합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ sudo apt update
	ubuntu@ubuntu-virtual-machine:~$ sudo apt install openjdk-8-jdk
	ubuntu@ubuntu-virtual-machine:~$ java -version
	~~~
	5. github에 있는 샘플 자바 어플리케이션을 clone 받기위해 git을 설치합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~$ sudo apt install git
	ubuntu@ubuntu-virtual-machine:~$ git -version
	~~~
	6. spring-music이라는 스프링 프로젝트를 clone 받은 후 gradle로 빌드합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace$ git clone https://github.com/cloudfoundry-samples/spring-music
	ubuntu@ubuntu-virtual-machine:~/workspace$ cd spring-music/
	ubuntu@ubuntu-virtual-machine:~/workspace/spring-music$ ./gradlew clean assemble 
	~~~
	7. cf push 명령으로 어플리케이션을 배포합니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/spring-music$ cf push
	~~~
	8. cf apps 명령으로 배포된 어플리케이션 목록을 조회할 수 있습니다. 조회된 urls 값으로 inception 서버에서 브라우저로 접속해볼 수 있습니다.
	~~~sh
	ubuntu@ubuntu-virtual-machine:~/workspace/spring-music$ cf apps
	Getting apps in org edu-org / space edu-space as admin...
	OK

	name           requested state   instances   memory   disk   urls
	spring-music   started           1/1         1G       1G     spring-music-sleepy-badger-ss.10.244.0.34.xip.io
	~~~
	9. 배포한 앱 정보 보기 : cf app spring-music
	10. 배포한 앱의 tail 로그 보기 : cf logs spring-music
	11. 배포한 앱의 최근 로그 보기 : cf logs spring-music --recent

# Cloud Foundry CLI 사용하기
* apps : 배포된 앱 목록을 조회하는 명령어
* services : 등록된 서비스를 조회하는 명령어
* marketplace : 마켓플레이스에 등록된 서비스 목록을 조회하는 명령어
* push : 앱을 manifest.yml에 명시된 정보를 기반으로 remote에 푸시하는 명령어
* create-user-provided-service(cups) : 마켓플레이스에 등록되지 않은 서비스에 대해서 사용자가 임의로 서비스를 등록할 수 있는 명령어
	1. 사용법 : cf create-user-provided-service SERVICE_INSTANCE [-p CREDENTIALS]
	~~~sh
	cf create-user-provided-service my-db -p '{"username":"admin", "password":"password"}'
	~~~
* env : push한 앱의 정보와 바인드 된 서비스 정보등을 조회할 수 있는 명령어
	~~~sh
	cf env welcome-cf  #welcome-cf 앱의 정보를 조회합니다.
	~~~
* ssh : push한 앱의 인스턴스로 ssh 접속할 수 있는 명령어. -L 속성을 이용하여 로컬호스트의 특정 포트로 터널링(포트 포워딩)도 가능합니다.
	~~~sh
	cf ssh welcome-cf -L 9999:10.0.1.14:3306  # 원격지의 10.0.1.14:3306을 로컬호스트의 9999번 포트로 터널링을 시도합니다.
	~~~

# Service Broker
* PaaS-TA의 마켓플레이스에 Service를 만들기 위해 필요한 장치입니다.

* 마켓플레이스에 Service를 생성하려면 Broker를 만들어서 사용해야 합니다.

* 기본적으로 Service Broker는 6개의 API가 기본적으로 구현되어야 합니다.

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