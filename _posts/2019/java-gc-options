# GC 로그 권장 옵션

```bash
JAVA_OPTS="$JAVA_OPTS -verbose:gc -XX:+PrintGCDateStamps"
JAVA_OPTS="$JAVA_OPTS -Xloggc:$APPLICATION_BASE/logs/gc_%p.log"
JAVA_OPTS="$JAVA_OPTS -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=2M"
```

## 기본 옵션

* `-verbose:gc` : GC 로그 활성화
* `-XX:+PrintGC` : 기본 출력(default)
* `-XX:+PrintGCDetails` : 상세 출력(일반적으로는 설정할 필요는 없음)
* `-XX:+PrintGCDateStamps` : `2019-03-05T11:40:24.831+0900` 추가
* `-Xloggc:$APPLICATION_BASE/logs/gc_%p.log` : 로그파일 경로 지정

## 파일 로테이션

* `-XX:+UseGCLogFileRotation` : GC로그파일 로테이션 On
* `-XX:NumberOfGCLogFiles=10` : 10개로 제한
* `-XX:GCLogFileSize=2M` : 최대 사이즈는 2M (8kb >=)

# Reference

* https://www.oracle.com/technetwork/articles/java/vmoptions-jsp-140102.html
  * 참고로 해당 문서에는 오타(`-` -> `+`)가 있으므로 주의를 요함(공식 문서인데...)
