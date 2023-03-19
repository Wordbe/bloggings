# Heap Dump



## 애플리케이션에서 메모리 과다 사용 문제

비정상적으로 힙메모리를 사용하고 있는 부분을 찾아 버그를 수정해야함.

- 힙메모리를 과다사용하면, GC 가 자주 발생해 애플리케이션 성능 저하가 되거나 OOM 으로 애플리케이션이 중단될 수 있다.

```bash
java.lang.OutOfMemoryError: Java heap space
	at ...
```

- Full GC 를 해도, 메모리가 큰 변화없다면 메모리 누수(memory leak) 가 발생하는 것이므로 힙 메모리 확인 필요하다.

<br />

<br />

## Heap 사용량 증가 원인

- 원인은 여러가지가 있겠지만 보통 개발자 코드가 의도하지 않은대로 작동해서 발생한 버그일 경우가 높다. 
- 혹은 많은 트래픽이 몰려와 평소보다 많은 힙 영역을 차지할 가능성도 있다.

<br />

<br />

<br />

## 애플리케이션 모니터링

1. GUI

- VisualVM (https://visualvm.github.io/)
  - OracleJDK 에서 제공하는 자바용 GUI 모니터링 툴이다.
- Heap size 이상을 발견 (평소보다 큼)

![](https://visualvm.github.io/images/visualvm_screenshot_20.png)

<br />

2. CLI

- jps: java 프로세스 조회

```bash
jps : 자바 프로세스 id 조회
63437 JavaTodoApplication

# 그 외 옵션
jps -l : 전체 패키지명 표시
jps -m : 메인메소드의 args 표시
jps -v : JVM 파라미터 표시
```

- jstat

```bash
# 사용량, 사용률, GC 수, GC 시간 등
jstat -gc [java-pid]
S0C         S1C         S0U         S1U          EC           EU           OC           OU          MC         MU       CCSC      CCSU     YGC     YGCT     FGC    FGCT     CGC    CGCT       GCT
      0.0      8192.0         0.0      5863.9      45056.0      24576.0      57344.0      25828.0    58688.0    58235.8    8512.0    8292.2     10     0.023     0     0.000     6     0.003     0.026

# 사용률, GC 수, GC 시간
jstat -gcutil [java-pid]
S0     S1     E      O      M     CCS    YGC     YGCT     FGC    FGCT     CGC    CGCT       GCT
0.00  71.58  54.55  45.04  99.23  97.42     10     0.023     0     0.000     6     0.003     0.026
```

- 각 영역(Survivor, Eden, Old 등)의 사용률이 너무 높지 않은지
- FGC(Full GC) 횟수가 너무 많지 않은지
- FGCT(Full GC Time) 시간이 너무 많지 않은지 확인

<br />

<br />

<br />

## heap dump

- jmap https://docs.oracle.com/javase/6/docs/technotes/tools/share/jmap.html

```bash
jmap -dump:format=b,file=heapdump.hprof [java-pid]
```

- OOM 발생시 자동으로 heap dump 를 생성하는 JVM 옵션도 있다.

```bash
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/heapdump # default 는 자바 프로젝트 최상위 경로
```

이렇게 `heapdump.hprof` 파일을 확보한다.

<br />

<br />

<br />

## 분석툴 사용

- MAT (Eclipse Memory Analyzer), VisualVM 등

**MAT**

- https://www.eclipse.org/mat/

- 힙덤프 분석시 기본 heap 크기: 1GB 이므로, 파싱시 OOM 발생

  MAT 툴이 할당할 수 있는 메모리가 heap dump 크기보다 크게 설정해주면 된다.

  ```bash
  vi /Applications/mat.app/Contents/Eclipse/MemoryAnalyzer.ini
  ...
  -Xms6G
  -Xmx6G
  ...
  ```

- 비정상적으로 높은 비율을 차지하는 객체를 확인

![](https://www.eclipse.org/mat/about/overview.png)

![](https://www.eclipse.org/mat/about/histogram.png)

<br />

<br />

<br />

<br />

