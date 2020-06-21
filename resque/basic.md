# resque
reque 는 백그라운드 작업을 작성해서 해당 작업을 여러 큐에 배치 한뒤, 나중에 처리하기 위한 루비기반 Redis 라이브러리  
실제로는 job 을 실행하는 rake task  
  
Resque 의 구성
- 작업 생성, 쿼리 및 처리를 위한 ruby 라이브러리
- task 를 처리하는  worker 를 시작하기 위한 rake job
- 대기열, 작업 및 작업자를 모니터링 하기 위한 Sinatra app
  
Resque 의 worker 들은 여러 머신에 분배되어 실행됨   
각 worker 들은 우선순위를 부여하고 메모리 누수/팽창을 핸들링 할 수 있으며 수행중인 작업을 알려주고 실페를 예상할 수 있음 
Resque 는 REE 에 최적되어 있음
ㄴ REE : MRI 및 JRuby 작업 
  
Resque 큐는 영구적이며  일정한 시간, atomic 한 push 와 pop(redis 를 이용) 이 가능하며    
job 을 json 패키지로 저장함으로써 시성을 제공힘   

Resque 는 이제 ruby 2.3.0 이상을 지원하며 Redis 3.0 이상을 지원함 

## 개요
Resque 는 작성된 job 을 queue 에 push 했다가 나중에 pull 해서 job 을 처리할 수 있다  
Resque 의 job 은 ruby class (or module) 로 perform 메소드를 수행한다  
```ruby
class Archive
  @queue = :file_serve

  def self.perform(repo_id, branch = 'master')
    repo = Repository.find(repo_id)
    repo.create_archive(branch)
  end
end
```
- @queue 는 Archive job 이 위치할 queue 를 나타냄  
- 원한다면 queue 에 이름을 지정할 수 도 있음  
  
Archive job 을 file_server 큐를 이용하려고 하면 우리는 어플리케이션에 이미 존재하는 Repository class 에 추가할 수 있음 
```ruby
class Repository
  def async_create_archive(branch)
    Resque.enqueue(Archive, self.id, branch)
  end
end
``` 
- async_create_archive 메소드를 호출하면 job 이 생성되고 file_server queue 에 job 이 배치됨
    
worker 는 아래의 코드를 실행하여 작업을 처리함     
```ruby
klass, args = Resque.reserve(:file_serve)
klass.perform(*args) if klass.respond_to? :perform
```
  
job 을 실행시키는 명령어
```shell script
$ cd app_root
$ QUEUE=file_serve rake resque:work
```
- resque worker 들이 1개씩 시작되어서 queue 에 작업이 추가됨  
- 시작할 준비가 되는대로 Resque.reserve 코드 snippet 이 실행되어 queue 를 반복적으로 polling 하면서 작업을 처리함
    
worker 에게 여러 queue 를 지정하고 여러 컴퓨터에서 실행할 수 있음  
Redis 서버에 접근 가능한 모든 서버에서 실행가능  

## install
resque gem 을 Gemfile 에 추가   
```Gemfile
gem 'resque'
```  
bundle 를 이용한 설치  
```shell script
$ bundle install
```

rake 파일 작성
- lib/tasks 내부에 rake 파일 작성
```ruby
require 'resque'
require 'resque/tasks'
require 'your/app
```


## worker 실행
Resque worker 는 무한루프 프로세스로 실행됨  
실행되는 구조는 아래와 같음  
```ruby
start
loop do
  if job = reserve
    job.process
  else
    sleep 5 # Polling frequency = 5
  end
end
shutdown
```  
  
worker 실행 명령어
```shell script
$ QUEUE=* rake resque:work
$ COUNT=2 QUEUE=* rake resque:workers
```

### 우선순위
숫자를 이용해서 우선순위를 지정할 수 없지만 queue 에 우선순위를 지정할 수 있음  
```shell script
$ QUEUES=critical,archive,high,low rake resque:work
```
위의 예제에서 critical 큐를 최우선으로 job 을 가져와 실행하고 critical 큐가 비었다면 archive 큐를 검사하여 job 을 실행할 수 있음  
  
우선순위 없이 실행 
```shell script
# 모든 큐는 공평하게 실행됨 
$ QUEUE=* rake resque:work
# 최상위 우선순위 제외하고 공평하게 실행 
$ QUEUE=critical,* rake resque:work
``` 

### 프로세스 ID
PIDFILE 옵션을 이용하여 PID 기반으로 프로세스에 접근할 수 있음 
```shell script
$ PIDFILE=./resque.pid QUEUE=file_serve rake resque:work
```

## frontend
Resque 에는 Sinatra 기반으로 frontend 가 제공되어 대기열의 상태를 확인할 수 있음  
```shell script
# 설정 파일을 이용하여 web 띄우기 
$ resque-web -p 8282 rails_root/config/initializers/resque.rb
```




## 래퍼런스 
- [resque github](https://github.com/resque/resque)
- [resque 소개 블로그](https://github.blog/2009-11-03-introducing-resque/)
