# ansible

## 설정 출력 
ansible-config view | grep roles_path


## 앤서블 컨셉
- control node
앤서블이 설치된 노드로 ad-hoc 과 playbook 명령을 실행 가능한 노드
/usr/bin/ansible, /usr/bin/ansible-playbook

- managed nodes
앤서블로 관리 될 서버 (호스트, agent)

- inventory
manged node 들의 리스트를 작성 (호스트들의 파일)



- modules
앤서블 코드를 실행하기 위한 앤서블의 코드 조각이자 인터페이스 
앤서블의 모듈을 이용하여 작업을 플레이북에 작성

- task
앤서블의 작업의 작은 단위 

- playbook
순서가 있는 작업 목록
작업과 변수가 포함될 수 있음 

## 개발 방법
vars 를 이용하여 변수의 그룹을 지정할 수 있음 
변수는 playbook, inventory 에서 사용 가능 

- ssh 통신 
ssh 키를 authorized_keys 를 시스템 파일에 추가해야함 

접속 할 user 은 -u 옵션을 이용해서 지정이 가능
환경 변수에 user 를 지정할 수 있음 

- 파일 전송
파일을 전송할 때, sftp 를 이용하여 통신 
sftp 를 지원하지 않으면 scp 를 이용해서 통신 

- 권한
명령어를 실행하기 위한 권한이 없는 경우 become 플래그를 이용
```sh
$ ansbile all -m ping -u bruce --become-user batman
```

## 모듈
```yml
- name: reboot server
  action: command /sbin/reboot -t now

- name: reboot server
  command: /sbin/reboot -t now
```

## extra-vars
command 에서 변수 전달해주기 

- 외부 변수
ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"

- 세션 변수
ansible-playbook site.yaml -i hostinv -e firstvar=false -e second_var=value2


https://stackoverflow.com/questions/30662069/how-can-i-pass-variable-to-ansible-playbook-in-the-command-line


## 플레이북 

### 플레이북 소개
[ansible document의 playbooks_intro](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)
[ansible document의 ansible example github](https://github.com/ansible/ansible-examples)

playbook 은 yaml 구문을 이용

- hosts: 명령을 실행할 호스트를 지정 
여러 호스트를 , 연결하여 지정 가능 

- order: 명령어가 실행될 호스트들을 순서를 제어 
    * inventory: 디폴트 값, inventory 에 지정된 순서로 실행
    * reverse_inventory
    * sorted: 호스트 이름으로 정렬하여 실행 
    * reverse_sorted
    * shuffle

- remote_user: 명령을 실행할 user 계정
ssh 로 접속할 리눅스 유저 
task 별로 지정하는 것도 가능함 

[권한 설정 become](https://docs.ansible.com/ansible/latest/user_guide/become.html)
- become: 
play or task 에서 권한을 갖고 실행하는 것 

- become_user
- become_method

- tasks: 각 play 에는 task 들이 포함되어 있음
playbook 실행하다가 실패한 호스트는 다음 작업들을 진행하지 않음 (제외됨) -> 플레이북 수정 필요 
각 task 의 목표는 모듈을 실행하는 것 
모듈들은 멱등성이 필요함 (항상 성공을 보장해야 함)
task 는 반드시 name 을 가져야 함 - 기본값은 action
name 은 플레이북이 실행될 때 출력하는 내용임 (확인할 내용...)
```
# 기본 형식
action: moudle [options]
```

변수를 사용 가능 - {{ vhost }} 

- handlers: 원격 시스템에서 변경이 발생할 떄, 헨들러를 이용해 지정 가능 
핸들러가 정의 된 위치에 따라서 global 핸들러를 지정할 수 있음 
이름이 같은 핸들러라고 존재하면 기존의 것을 덮어 씀 
핸들러는 pre_tasks, tasks, post_tasks 에서 notify 영역에서 지정 
roles 내부의 핸들러는 자동으로 tasks 의 마지막에 플러시 됨 

핸들러는 include_role, import_role 에서 다르게 동작 
https://medium.com/opsops/include-role-import-role-and-handlers-in-ansible-b32a5386a555

- notify: 이 영역에 핸들러들을 지정하여 호출 

```yml
- name: template configuration file
  template:
    src: template.j2
    dest: /etc/foo.conf
  notify:
     - restart memcached
     - restart apache

handlers:
    - name: restart memcached
      service:
        name: memcached
        state: restarted
    - name: restart apache
      service:
        name: apache
        state: restarted
```

- include_vars: 값을 로드 
```yml
tasks:
  - name: Set host variables based on distribution
    include_vars: "{{ ansible_facts.distribution }}.yml"

handlers:
  - name: restart web service
    service:
      name: "{{ web_service_name | default('httpd') }}"
      state: restarted
```

- 플레이북 실행
```sh
# 10 개 병렬로 플레이북 실행 
$ ansible-playbook playbook.yml -f 10
```

### 앤서블 플레이북은 정적 or 동적 모드가 있음 
[ansible document의 재사용 가능한 playbook](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html)

동적 모드: 런타임 중에 처리됨

https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_includes.html
- import: 플레이북이 파싱되기 전에 전처리됨 
- include: 해당 플레이북을 실행할 때 처리됨 

[include와 변수](https://shasawas.wordpress.com/2016/05/23/how-to-loop-over-a-set-of-tasks-in-ansible/)


include
- include_tasks
- include_role

import
- import_playbook
- import_task



[ansible document의 playbook keywords](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html)

### play

https://taesany.tistory.com/140
tags: 태그를 사용하면, 작업의 하위 집합을 선택할 수 있음 

동일한 tag 는 같은 그룹의 개념으로 묶어서 해당 tag 가 붙은 것들만 실행 
tag 중 실행하고 싶지 않은 것은 --skip-tags=태그이름 


```shell
# tag 실헹
ansible-playbook [-t | --tags 태그이름] 
# tag 확인 
ansible-playbook --list-tags
```

tag 를 role 적용 
- { role: common, tags: ['common']}

https://stackoverflow.com/questions/54085972/running-an-ansible-playbook-on-localhost-but-referring-to-inventorys-group-var


- serial: 한 번에 실행할 호스트 수를 지정
ex) 총 호스트 수 : 10, serial: 2 로 지정하면 
-> 2 호스트 씩 끊어서 play 를 실행 

https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html


### role
http://oniondev.egloos.com/9978477
https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html

Role 은 여러개의 연관된 Task 를 체계화 하고, 그에 쓰이는 데이터를 묶는데 좋다


기본 디렉토리 구조 
roles
rolename
- defaults: 기본 변수들 (기본 값 정의, 입력 값 보다 우선순위가 낮음)
- files: 배포할 파일들 
- handlers: role 에서 사용할 핸들러 
- meta: 메타 데이터 
- templates: 배포할 템플릿 
- tasks: 주요 작업 목록
- vars: 변수들 
각 폴더 안에서 Ansible 은 main.yml 파일을 자동으로 읽음


앤서블 roles 예제 
[](https://github.com/ansible/ansible-examples/blob/master/tomcat-memcached-failover/roles/common/tasks/main.yml)
```yml
# https://centos.pkgs.org/7/centos-x86_64/libselinux-python-2.5-14.1.el7.x86_64.rpm.html
# https://pypi.org/project/selinux/
# 개발을 위한 python 바인딩이 포함됨 (SELinux 응용 프로그램)
- name: Install libselinux-python
  yum: name=libselinux-python state=present

# http://faq.hostway.co.kr/Linux_ETC/7095
# EPEL(Extra Packages for Enterprise Linux)은 Fedora Project에서 제공하는 각종 패키지의 최신 버전을 제공하는 community 기반의 저장소
# RHEL 은 패키지 업데이트가 거의 없음
# 최신 버전의 패키지를 사용하기 위해서 EPEL 이나 REMI 등의 레파짓토리를 이용
- name: Install GPG key for EPEL
  get_url: url=https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-6 dest=/etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

- name: Install EPEL repository
  yum: name=https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm state=present

# 
- name: Setup Iptables rules
  template: src=iptables.j2 dest=/etc/sysconfig/iptables
  notify: restart iptables
```

include_role, import_role 을 이용하여 인라인으로 사용할 수 있음 

meta/main.yml 에 dependencies 에 role 종속성을 기술 할 수 있음 


### block
[ansible document의 playbooks_blocks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_blocks.html)
[jeffgeerling features-ansible-20-blocks](https://www.jeffgeerling.com/blog/new-features-ansible-20-blocks)
작업을 논리적으로 그룹화하고 오류를 처리 가능
루프를 제외하고 대부분의 내용을 블록에 정의하여 사용 가능 
매개변수를 block 수준에서 관리 가능 

when 과 block 을 이용한 예제
```yml
---
- hosts: web
  tasks:
    # Install and configure Apache on RedHat/CentOS hosts.
    - block:
        - yum: name=httpd state=installed
        - template: src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
        - service: name=httpd state=started enabled=yes
      when: ansible_os_family == 'RedHat'
      sudo: yes

    # Install and configure Apache on Debian/Ubuntu hosts.
    - block:
        - apt: name=apache2 state=installed
        - template: src=httpd.conf.j2 dest=/etc/apache2/apache2.conf
        - service: name=apache2 state=started enabled=yes
      when: ansible_os_family == 'Debian'
      sudo: yes
```

with_items, when 과 함께 쓰는 경우 매우 편리함 

특정 작업에서 장애를 처리하는 경우도 매우 유용함
```yml
---
tasks:
  - block:
      - name: Shell script to connect the app to a monitoring service.
        script: monitoring-connect.sh
    rescue:
      - name: This will only run in case of an error in the block.
        debug: msg="There was an error in the block."
    always:
      - name: This will always run, no matter what.
        debug: msg="This always executes."
```
- blcok 을 먼저 실행하고 실패하면 rescue 를 실행 
- always 는 항상 실행되는 구문 
(try, catch, finally)

장애를 처리할 때, failed_when 을 사용하는 것이 일반적으로 좀 더 간편함 

### task



changed_when: tasks의 changed 상태를 재정의하는 조건식

environment: 실행시 task 에 제공할 환경변수로 변환되는 딕셔너리 


## 앤서블 playbooks_best_practices
[ansible document의 playbooks_best_practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)


## 조건문
when 

특정 호스트에 대해서 작업을 건너뛰거나
특정 패키지가 없거나
해당 운영체제가 지원 버전보다 낮거나
파일 시스템이 꽉 차거나 

복합 조건
```yml
tasks:
  - name: "shut down CentOS 6 and Debian 7 systems"
    command: /sbin/shutdown -t now
    when: (ansible_distribution == "CentOS" and ansible_distribution_major_version == "6") or
          (ansible_distribution == "Debian" and ansible_distribution_major_version == "7")

# 두 조건이 and 인 경우 
tasks:
  - name: "shut down CentOS 6 systems"
    command: /sbin/shutdown -t now
    when:
      - ansible_distribution == "CentOS"
      - ansible_distribution_major_version == "6"
```

register 변수
플레이북에서 실행한 명령의 결과를 변수로 저장했다가 나중에 재사용하기 위해서 사용 
어느 변수에 실행 결과를 저장할 것인지를 지정 

```yml
- name: registered variable usage as a with_items list
  hosts: all

  tasks:
      - name: retrieve the list of home directories
        command: ls /home
        register: home_dirs # 등록

      - name: add home dirs to the backup spooler
        file: path=/mnt/bkspool/{{ item }} src=/home/{{ item }} state=link
        with_items: "{{ home_dirs.stdout_lines }}"
        # same as with_items: "{{ home_dirs.stdout.split() }}"
        # ls /home 결과 list 에 같은 동작을 적용 
```

## 반복문 

## ansible 변수 우선순위 
https://stackoverflow.com/questions/29127560/whats-the-difference-between-defaults-and-vars-in-an-ansible-role

- extra_vars: 외부 변수 우선
- 인벤토리에 정의된 연결 변수: ansible_ssh_user, etc
- 일반적인 변수: play 속 vars, include 된 vars, role 의 vars
- 인벤토리의 정의된 일반 변수 
- 시스템이 발견한 fact 변수
- role 의 defaults 변수 

## group_vars
http://theeye.pe.kr/archives/2594

인벤토리에 변수들을 직접 저장하지 않고 그룹 변수를 이용하는 것이 좋음 - 인벤토리는 변하는 것 
-> 입력과 내부 변수의 분리 

## 인벤토리 
https://stackoverflow.com/questions/33126156/is-it-possible-to-define-playbook-global-variables-in-ansible

- all 은 호스트 파일의 모든 그룹을 포함 
- 인벤토리 내부에 group_vars 와 동일한 디렉토리 레벨에서 디렉토리의 이름을 따서 별도의 파일에 포함할 수 있음 



https://stackoverflow.com/questions/17188147/how-to-run-ansible-without-specifying-the-inventory-but-the-host-directly
inventory 는 파일이 아닌 명령어에서 전달 가능 

$ ansible-playbook -i example.com, playbook.yml

ansible-playbook playbook/oauth_webapp_deploy.yml -i ka-build-weblogin-test-1.ay1.krane.9rum.cc


## var_files vs include_vars
https://stackoverflow.com/questions/53253879/ansible-vars-files-vs-include-vars



## yum 모듈 
[](https://docs.ansible.com/ansible/latest/modules/yum_module.html#yum-module)
- name: 설치할 라이브러리 
URL, 로컬 경로를 rpm 에 등록해서 사용하는 경우, state = present 사용 

- state: 명령의 상태 
설치를 위한 present, installed, latest
삭제를 위한 absent, removed

present, installed 는 해당 패키지가 설치 되어 있는지 확인함 
latest 는 현재 버전이 최신이 아니면 업데이트 함 

https://stackoverflow.com/questions/40410270/what-is-the-difference-between-two-state-option-values-present-and-install
present 와 installed 는 서로 alias 지만 installed 는 2.9 에서 deprecated

기본 값은 None 이지만, 
autoremove 가 yes 이면 absent 
autoremove 가 no 이면 present



## return values  
[](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html)

- backup_file: 
- changed:
- failed:
- invocation:
- msg:
- rc:
- results:
- skipped
- stderr:
- stderr_lines:
- stdout:
- stdout_lines:


- ansible_facts:
- exception:
- warnings:
- deprecations:

## ansible 모듈 
### git
https://docs.ansible.com/ansible/latest/modules/git_module.html?highlight=git
- accept_hostkey: ssh 옵션으로 ssh 의 "-o StrictHostKeyChecking=no" 를 실행한 것과 같음 
http://taewan.kim/post/ssh_config/
StrictHostKeyChecking 는 ~/.ssh/known_hosts에 자동으로 호스트를 추가하는 설정
StrictHostKeyChecking=no 로 지정하면 호스트를 자동으로 추가하지 않음 
- force: local 저장소의 내용을 orgin 으로 완벽히 하는 거 


### shell
https://docs.ansible.com/ansible/latest/modules/shell_module.html
- creates: 파일 이름이 이미 존재하면이 단계는 실행 되지 않습니다
- cmd: 실제로 실행할 명령어 

### agrs
args 는 task 단계에서 사용하는 것으로 해당 모듈에 변수를 전달
ex) shell 에서 creates 속성을 agrs 로 지정할 수 있음
```yml
shell: 
args:
  creates: 
```

### get_url
https://docs.ansible.com/ansible/latest/modules/get_url_module.html


## check_mode
https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html


## 플레이 북에서의 오류 처리
https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html

- ignore_errors 
명령 실패를 무시하는 것으로 기본적으로 에러 발생하면 play 가 중지되는 것을 방지 

- --force-handlers
호스트에서 task 가 실패하면 이전에 notify 된 핸들러는 해당 호스트에서 실행되지 않음
그래서 관련이 없는 장애가 호스트를 예상할 수 없는 상태로 유지할 수도 ...
나중에 같은 플레이에서 작업이 실패해도 handler 가 동작하지 않음 .. 
--force-handlers 옵션을 사용해서 handler 를 강제시킬 수 있음

- failed_when
각 작업의 실패의 의미를 정의 

- changed_when
변경된 결과를 재정의 하는 것
일반적으로 명령어가 실행되면 시스템 상태에 영향을 미치는지에 따라서 change 가 표현됨
변경사항이 없도록 해서 handler 가 동작하지 않게 할 수 있음 

- any_errors_fatal
호스트 나머지 작업을 중단시킬때 사용

- block, rescue 
루프가 아니라면, block 내부의 내용을 실패하면 rescu 에서 실행할 수 있음 

## Facts
http://oniondev.egloos.com/9978729

Playbook 을 실행하였을때 항상 처음에 "gathering facts" 라고 나옴
앤서블은 실행하기 전에 시스템의 환경정보를 모으는데, 이것을 facts 라고 함
- 환경 정보: cpu core 개수, 네트워크 IPv4, IPv6, 디스크 마운트 정보, 리눅스 버전 등 

이러한 환경 정보를 이용해서 task, template 에서 사용 가능
ex) nginx 인스턴스를 cpu 개수만큼 띄우기 

## Vault
https://docs.ansible.com/ansible/latest/user_guide/vault.html

앤서블 template 에 민감 정보를 저장해야 할 수 있음 
ex) ssh key 라던가 ... (톡 서버 파트)

Valut 는 yaml 파일을 암호화할 때 사용할 수 있음 
암호화된 파일을 만들때, 패스워드를 입력해서 따로 관리 해야함 


- 암호화 하기
ansible-vault encrypt [파일이름]

원래 파일 자체가 암호화 됨 
패스워드 입력 .. 

- 복호화 후 출력 
ansible-vault view [파일이름]

패스워드 입력하면 출력됨 

- 복호화 하기 
ansible-vault decrypt [파일이름]


- 암호화 파일 만들기 
ansible-vault create [파일이름]

- 패스워드 수정하기 
ansible-vault rekey [파일이름]

- 패스워드 입력으로 .. 
--ask-vault-pass

## hasivault 플러그인
hashicorp 에서 제공하는 vault 사용
사내 vault 와 연동가능 

[예제](https://www.serverlab.ca/tutorials/dev-ops/automation/using-hashicorp-vault-with-ansible-jinja2-templates/)
[앤서블 문서](https://docs.ansible.com/ansible/latest/plugins/lookup/hashi_vault.html)

## lookup
lookup 플러그인을 이용하면 외부 데이터 소스에 액세스 할 수 있음 

외부 플러그인도 사용 가능해짐 


```yaml
vars:
  motd_value: "{{ lookup('file', '/etc/motd') }}"
tasks:
  - debug:
      msg: "motd value is {{ motd_value }}"
```

https://docs.ansible.com/ansible/latest/user_guide/playbooks_lookups.html

## 앤서블 환경 설정 
https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings

ansible.cfg 는 명령어를 실행하는 디렉토리로 이동해서 실행해햐 함 ... 

- forks: 앤서블이 병렬 실행할 수를 지정 



## 이전 설치 검사
- shell 사용시 creates 속성을 이용하여 해당 파일이 존재하지 않을 때만 실행하도록
- register, when 을 이용하여 설치 여부를 확인 하는 방법 


# delegate_to
https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html

delegate_to 를 이용하여 다른 호스트를 지정하여 수행 
- 127.0.0.1 처럼 

run_once 를 지정하여 1번만 실행 

# uri 

https://docs.ansible.com/ansible/latest/modules/uri_module.html

# synchronize
https://docs.ansible.com/ansible/latest/modules/synchronize_module.html
rsync 의 래퍼 모듈 

- rsync: remote sync
https://www.lesstif.com/pages/viewpage.action?pageId=12943658

file & directory 동기화를 위한 유닉스 유틸리티 
대역폭 최소화를 위해서 delta encoding algorithm 을 구현 
rcp, scp 보다 훨씬 빠르고 효율적 

클라이언트와 서버가 존재하며, 서버는 873 포트를 사용 
```shell
# rsync options source destination 
# -v: verbose 정보 출력
# -r: 재귀적으로 하위 디렉터리까지 복사 , 전송시 타임 스탬프나 퍼미션을 보장하지 않음 
# -a: archive mode
#   -t(타임 스탬프 보존), -l(심볼릭 링크 보존), -p(퍼미션 보존), -g(그룹 보존), -o(소유자 보존: root만), -D(device special 보존)
# -z: 데이터 압축
# -h: human-readable 
```

## Test
--check 옵션을 이용하여 테스트 실행가능 



curl 'http://api.noti.daumkakao.io/send/group/kakaotalk' -d 'to=8562' -d "msg=LaputaAccountApp 빌드에 실패하였습니다. 확인 부탁드려용~ ${JENKINS_URL}/job/laputa-account-app/"


## ansible localhost
https://gist.github.com/alces/caa3e7e5f46f9595f715f0f55eef65c1
