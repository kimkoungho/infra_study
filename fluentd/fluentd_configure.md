# 설정 분석하기 

## 설정 파일의 위치 
- td-agent 를 이용한 설치  
설정 파일은 /etc/td-agent/td-agent.conf 에 위치함  
ㄴ 커스텀 디렉토리를 만들어서 include 하는 방식으로도 사용할 수 있음  

- gem 을 이용한 설치 
gem 을 이용해서 설치한 경우 아래의 명령을 사용해서 설정 파일을 작성 가능 
```shell script
$ sudo fluentd --setup /etc/fluent
$ sudo vi /etc/fluent/fluent.conf
``` 

## 설정 파일의 구성 
- source: 입력 소스를 지정 
- match: 출력할 목적지를 지정 
- filter: 이벤트의 전처리 파이프 라인 지정
- system: 시스템의 전체 설정을 세팅
- label: 내부 라우팅을 위한 filter 와 output 의 그룹을 나타냄 
- @include: 다른 파일을 include 할 때 사용 

### source
source 를 이용하여 원하는 입력 플러그인을 선택하고 구성함  
```editorconfig
# Receive events from 24224/tcp
# This is used by log forwarding and the fluent-cat command
<source>
  @type forward
  port 24224
</source>

# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>
```
- 각 source 에는 @type 이라는 매개변수가 필수적으로 있어야 함  
- @type 에 forward, http 를 설정하여 tcp, http 등 프로토콜을 사용가능  

### match 
일치하는 태그가 있는 이번트를 찾아서 처리하는 것으로 가장 일반적으로 사용하는 용도는 다른 시스템에 이벤트를 출력하는 것  
ㄴ match 는 출력 플러그인이라고 함  

```editorconfig
# Receive events from 24224/tcp
# This is used by log forwarding and the fluent-cat command
<source>
  @type forward
  port 24224
</source>

# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>

# Match events tagged with "myapp.access" and
# store them to /var/log/fluent/access.%Y-%m-%d
# Of course, you can control how you partition your data
# with the time_slice_format option.
<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```
- match 에는 일치하는 패턴과 매개변수를 포함해야 됨  
ㄴ myapp.access 태그가 있는 이벤트만 출력 대상으로 전송  
- @type 으로 출력할 대상을 지정  

### filter
filter 를 사용해서 전처리가 가능함  
```
입력 -> 필터 1 -> 필터 2 ... -> 필터 N -> 출력
```
record_transformer 라는 표준 필터 추가  
```editorconfig
# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>

<filter myapp.access>
  @type record_transformer
  <record>
    host_param "#{Socket.gethostname}"
  </record>
</filter>

<match myapp.access>
  @type file
  path /var/log/fluent/access
</match>
```
- 수신된 이벤트는 filter 의 패턴에 일치하면 필터로 이동함
예를들어 {"event": "data"} 가 입력으로 들어온 경우  
- host_param 필드를 이벤트에 추가하고 필터링 된 이벤트가 출력(match) 로 이동함
- 출력으로 이동하는 데이터는 {"event": "data", "host_param": "webserver1"} 이 됨  
  
### system
시스템 전체 구성을 설정할 때 사용하는 지시문으로 대부분 CLI 를 이용해서 설정 가능함  
- log_level
- suppress_repeated_stacktrace
- emit_error_log_interval
- suppress_config_backup
- without_source
- process_name


### label
내부 라우팅을 위해 필터 및 출력을 그룹화  
label 을 이용하면 태그 처리의 복잡성을 줄일 수 있음  
```editorconfig
<source>
  @type forward
</source>

<source>
  @type tail
  @label @SYSTEM
</source>

<filter access.**>
  @type record_transformer
  <record>
    # ...
  </record>
</filter>
<match **>
  @type elasticsearch
  # ...
</match>

<label @SYSTEM>
  <filter var.log.middleware.**>
    @type grep
    # ...
  </filter>
  <match **>
    @type s3
    # ...
  </match>
</label>
```
- label 은 내장 플러그인의 매개변수이므로 @ 이라는 prefix 가 필요함 
- forward 이벤트는 record_transformer 필터 / elasticsearch output 에 라우팅 됨
- tail 이벤트는 grep 필터 / s3 output 에 라우팅 됨  
  
**@ERROR label**  
emit_error_event 플러그인 API 에서 생성한 오류 레코드에 사용되는 내장 label  
<label @ERROR> 를 설정한 경우 버퍼가 가득차거나, 유효하지 않은 레코드와 같은 오류가 발생할 때, 해당 label 로 라우팅됨  


### @include
해당 지시문을 사용하여 별도의 설정 파일을 가져올 수 있음  
```editorconfig
# absolute path
@include /path/to/config.conf

# if using a relative path, the directive will use
# the dirname of this config file to expand the path
@include extra.conf

# glob match pattern
@include config.d/*.conf

# http
@include http://example.com/fluent.conf
```

## pattern 
<match>, <filter> 에 와일드 카드를 사용할 수 있음    
- *: 단일 태그 부분과 일치 
    * a.*  은 a.b 는 일치 
    * a.*  은 a, a.b.c 와는 일치하지 않음 
- **: 0 개 이상의 패턴과 일치 
    * a.** 은 a, a.b, a.b.c 와 일치 
- {X, Y, Z}: X or Y or Z 와는 일치함을 표현  
- \#{...}: 괄호 안의 문자열을 Ruby 표현식으로 평가함  

## 지원되는 값 유형
- string: 필드가 문자열로 분석됨 (', ")
- integer: 필드가 정수로 분석됨
- float: 필드가 float 분석됨
- size: 필드가 바이트 수로 분석됨  
    * <Integer>k 는 킬로바이트 
    * <Integer>m 은 메가바이트
    * <Integer>g 는 기가바이트 
    * <Integer>t 는 테라바이트
    * 기본값은 byte  
- time: 필드가 시간으로 분석됨 
    * <Integer>s 는 초 
    * <Ingeger>m 은 분
    * <Integer>h 는 시간 
    * <Integer>d 는 일
    * 기본 값은 초 이며 0.1 = 100ms 임  
- array: json 배열로 분석됨 
    * 표준: ["key1:, "key2"]
    * 비표준: key1,key2
- hash: json 객체로 분석됨 

## 래퍼런스 
- [fluetnd config-file](https://docs.fluentd.org/configuration/config-file)
