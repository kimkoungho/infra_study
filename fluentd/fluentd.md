# Fluentd
Fluentd 는 C 와 Ruby 로 작성된 여러 소스에서 발생한 다양한 포맷의 데이터를 단일 포맷(Json) 으로 리포맷 가능한 로그 수집기 중 하나  
Fluentd 는 총 7개의 컴포넌트로 구성됨  
- Input
- Parser
- Engine
- Filter
- Buffer
- Output
- Formatter
각 컴포넌트 마다 제공되는 플러그인을 적절하게 활용하여 사용자 환경에 맞도록 데이터를 수집, 파싱, 저장 가능함  
ㄴ 사실 저장은 별도의 저장소를 두는게 일반적임 (elastic search, hadoop)  
- [Fluentd 플러그인 공식](https://docs.fluentd.org/)

## 

목적 
- Fluentd 라는 로그 수집기를 활용하여 Object Storage 에 저장하는 과정 

- td-agent: 루비로 작성된 fluentd 를 쉽게 사용할 수 있게 해주는 래퍼 프로그램


## Reference
- [네이버 클라우드 플랫폼 medium](https://medium.com/naver-cloud-platform/%EC%9D%B4%EB%A0%87%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%98%EC%84%B8%EC%9A%94-fluentd-%ED%99%9C%EC%9A%A9%ED%95%98%EC%97%AC-object-storage%EC%97%90-%EB%A1%9C%EA%B7%B8-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0-7b2f55c671c6)

