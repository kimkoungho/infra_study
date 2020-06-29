# curl


## curl 을 이용한 localhost 호출시 dns 적용
종종 nginx 를 태우기 위해서 localhost 를 dns 로 호출하는 경우가 있음  
이런 경우 --resolve 옵션을 이용하여 호출 가능  
```shell script
$ curl --resolve 'test.com:80:127.0.0.1' http://test.com/something
``` 

[참고한 stackoverflow](https://stackoverflow.com/questions/3390549/set-curl-to-use-local-virtual-hosts)

--resolve 옵션 :
