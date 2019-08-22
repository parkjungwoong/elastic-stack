#자주 쓰는 grok 패턴

## 목차
- [일반적인 log4j](#일반적인-log4j)
- [로그 내용의 시간을 타임스탬프 값으로 사용](#로그-내용의-시간을-타임스탬프-값으로-사용)
- [파일명의 날짜를 가져올 때](#파일명의-날짜를-가져올-때)

## 일반적인 log4j
- 로그
```
[2019-08-22 23:08:18][REQID12345][INFO] testService.selectTest(...): test log
```
- Grok 패턴
``` json
grok {
   match => { "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\]\[%{WORD:thread_id}\]\[%{LOGLEVEL:log_level}\]\s%{GREEDYDATA:message}"}
}
// '[' 문자 같은 경우 '\' 백슬래시 문자를 붙여줘야함
// 단어 사이에 공백이 있을 경우: \s, 공백이 있거나 없을 경우: \s{0,1}
```
- 파싱 결과
``` json
{
  "timestamp": "2019-08-22 23:08:18",
  "thread_id": "REQID12345",
  "log_level": "INFO",
  "message": "testService.selectTest(...): test log"
}
```

## 로그 내용의 시간을 타임스탬프 값으로 사용
- 로그
```
2019-08-22 23:08:18 ............
```
- Grok 패턴
``` json
grok {
   match => { "message" => "%{TIMESTAMP_ISO8601:timestamp}........"}
}
//message 필드 내에 시간 값을 파싱 후 타임스탬프 값으로 사용
date {
  match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
}
```

## 파일명의 날짜를 가져올 때
- 로그
```
[23:08:18] 로그 내용
// 이렇게 로그 내용내 날짜 정보가 없고 debug.log.2019-08-22 이것 처럼 파일명에 날짜 정보가 있을 경우, 파일명의 날짜를 가져와야함 
```
- Grok 패턴\
아래 예시는 filebeat 7.x버전대의 전송값으로 작성함, filebeat로 logstash로 받을 경우 file에 대한 정보도 같이 보냄을 이용해 날짜 정보 파싱
``` json
//message: '...', log : { file : { path : '/log/api/debug.log.2019-08-22' } ..} ..}, .. 이런식으로 전송되었을 때
grok {
   match => { "message" => "%{TIME:time}........"}
}
ruby {
  code => 'event.set("timestamp", event.get("[log][file][path]").split("/").last.split(".").last[0..9]+" "+event.get("time"))'
}
```

