# JIT, AOT Compiler



## JIT

- Just In Time Compilation
- Java HotspotVM 의 기본설정.
- 런타임에 java 바이트코드를 기계어로 변환한다.
- 즉 동적 컴파일을 함.
- java 메소드를 미리 수천번 실행하여 워밍업한다. 이를 통해 컴파일러가 전체 클래스 구조를 알고 최적화할 수 있다.



### C1 Compiler

- some value numbering, 인라이닝, 클래스 분석을 수행하는 컴파일러
- 빠르고 가볍게 최적화

### C2 Compiler

- global value numbering, method inlining, intrinsic replacement, 루프 변환 등을 수행하는 컴파일러



## AOT

- Ahead Of Time Compilation, 소스코드를 미리 컴파일
- GraalVM 에서 지원
- 빌드타임에 java 바이트코드를 기계어로 변환한다.
- 애플리케이션 실행시 적은 오버헤드로 빠르게 실행된다.
- 워밍업 없어서 초기 실행 더 빨라짐





컴파일러

- constant inling (인라이닝)
- loop unrolling (반복문 펼치기)
- partial evaluation (부분 평가)
- escape analysis (탈출 분석)



