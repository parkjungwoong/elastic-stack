# kibana 플러그인 개발 환경 구성

> 테스트 환경 정보 kibnan 7.2, elasticsearch 7.2

## 목차
- [Kibana 개발 환경 설정](#kibana-개발-환경-설정)
- [플러그인 개발 환경 설정](#플러그인-개발-환경-설정)
- [환경 설정 트러블 슈팅](#환경-설정-트러블-슈팅)

## kibana 개발 환경 설정
- kibana repo clone
    ```
    git clone https://github.com/elastic/kibana.git
    ```
- 의존성 파일 다운로드
    ```
    cd kibana
    #check node version
    #nvm use - 노드 버전 변경 필요
    #yarn 없으면 설치 필요, npm install -g yarn
    yarn kbn bootstrap
    ```
- elasticsearch 연동 설정

    여기서는 외부 el 사용, 로컬 el설정은 [여기](https://github.com/elastic/kibana/blob/master/CONTRIBUTING.md#running-elasticsearch-locally) 참고
    ```
    vi config/kibana.yml
    
    elasticsearch.hosts:
      - {{ url }}
    elasticsearch.username: {{ username }}
    elasticsearch.password: {{ password }}
    elasticsearch.ssl.verificationMode: none
    ```
- run server, 설정 완료 테스트
    ```
    yarn start
    ```
    
## 플러그인 개발 환경 설정
- 템플릿 생성
    ```
    cd kibana
    node scripts/generate_plugin 플러그인 이름
    #prompt 출력나오는거에 따라 생성
    ```
- 플러그인 실행
    ```
    cd plugins/플러그인명
    yarn start
    ```
    
- 환경 설정 트러블 슈팅

    에러 문구 : 
    ```
    Error: spawnSync node ENOENT
        at Object.spawnSync (internal/child_process.js:990:20)
        at spawnSync (child_process.js:601:24)
        at execFileSync (child_process.js:629:13)
        ....
    ```
    kibana root 경로를 지정해줘야됨
    
    ```
    vi /kibana/packages/kbn-plugin-helpers/lib/plugin_config.js
        
    return Object.assign(
        {
          root: root,
          //kibanaRoot: pkg.name === 'x-pack' ? resolve(root, '..') : resolve(root, '../../kibana'),
          kibanaRoot: '키바나 root 경로 지정',
          serverTestPatterns: ['server/**/__tests__/**/*.js'],
          buildSourcePatterns: buildSourcePatterns,
          .....
          
       }
    )
    ```