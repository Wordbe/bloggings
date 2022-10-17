# Kotlin으로 서버 개발하기 1



개발하다보면 조금 더 간결한 표현 방법은 없는지 많이 고민하게 됩니다.

나도 동료도 코드를 더 쉽게 읽을 수 있다면 덜 피곤해지기 때문이죠.



이번 글에는 코틀린 서버 개발 세팅을 진행합니다.

프로젝트를 만들면서 진행합니다.



## 1. 프로젝트 생성

![](https://i.ibb.co/VwYWd5s/2022-09-28-01-50-49.png)

- 인텔리제이 Ultimate 버전입니다. 무료로 진행하려면 https://start.spring.io/ 에 방문하셔서 zip 파일 형태로 프로젝트를 생성할 수 있습니다.
- 언어: kotlin, Java 17
- 빌드 및 의존성 도구: gradle



## 2. 의존성 추가

리액티브 웹 앱 생성을 위해 아래의 의존성을 추가합니다.

- 리액티브로 웹 앱 간단한 예제 프로젝트입니다. (Web MVC 로 하셔도 됩니다.)
- 이 글의 초점은 코틀린을 활용한 웹 서버 개발입니다.
- 프로젝트 생성시 아래와 같은 라이브러리를 추가했습니다.
  - Spring Reactive Web
  - Spring Data R2DBC
  - H2 Database



**build.gradle.kts**

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    id("org.springframework.boot") version "2.7.4"
    id("io.spring.dependency-management") version "1.0.14.RELEASE"
    kotlin("jvm") version "1.6.21"
    kotlin("plugin.spring") version "1.6.21"
}

group = "co.wordbe"
version = "0.0.1-SNAPSHOT"
java.sourceCompatibility = JavaVersion.VERSION_17

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-data-r2dbc")
    implementation("org.springframework.boot:spring-boot-starter-webflux")

    implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
    
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")

    runtimeOnly("com.h2database:h2")
    runtimeOnly("io.r2dbc:r2dbc-h2")

    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("io.projectreactor:reactor-test")
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "17"
    }
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

여기서 주목해야할 점은 기존 자바에서 사용하는 기능을 코틀린에서도 사용할 수 있도록 의존성이 추가되는 부분입니다.

```kotlin
implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
implementation("org.jetbrains.kotlin:kotlin-reflect")
implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
```

- jackson, reflect, jdk8 기능이 호환되도록 한다고 추정할 수 있습니다.

<br />

```kotlin
implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
```

- 마찬가지로, 리액티브 웹 기능을 추가했더니 위 의존성을 통해 project reactor (리액티브 스트림 구현체), 코틀린의 coroutine 기능을 자연스럽게 사용할 수 있도록 추가되었습니다.



<br />

## 3. 프로젝트 실행

**PomodoroApplication.kt**

```kotlin
package co.wordbe.pomodoro

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class PomodoroApplication

fun main(args: Array<String>) {
    runApplication<PomodoroApplication>(*args)
}
```

- 기존에 자바에서 보던 것과 상당히 유사한 메인 클래스입니다.
- 코틀린은 한 파일 안에 여러 클래스와 여러 함수를 담을 수 있습니다.
- 위에서는 클래스와 함수를 분리시켜 각각 top-level 에 위치시켰습니다.
  - 클래스에는 어노테이션이 붙어 스프링부트에 필요한 빈을 탐색해서 스프링 컨테이너에 등록해 놓을 것이고,
  - 메인은 말그대로 애플리케이션을 구동하는 역할을 하는 함수입니다.
- cmd + shift + R 로 잘 실행이 되는 것을 확인할 수 있습니다.



