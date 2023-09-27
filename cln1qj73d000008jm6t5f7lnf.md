---
title: "Version Catalog + Convention Plugin으로 build.gradle 버전을 관리해보자!"
datePublished: Wed Sep 27 2023 12:41:11 GMT+0000 (Coordinated Universal Time)
cuid: cln1qj73d000008jm6t5f7lnf
slug: version-catalog-convention-plugin-buildgradle
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695720599157/07ac1a1a-88a6-4789-bd7a-7ac5ec7edfa0.png
tags: android, version-catalog, convention-plugin, build-gradle, multi-module

---

## Why Version Catalog, Convention Plugin?

Multi Module의 버전을 관리하는 방법은 크게 세가지(ext, buildSrc, version catalog)가 있다. 각각의 장단점을 알고 싶다면 [여기](https://jinukeu.hashnode.dev/android-multi-module)를 참고하면 된다. (대충 version catalog가 가장 좋다는 얘기)

## Version Catalog 적용 순서

### 1\. libs.versions.toml 파일 생성

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695803396232/ed0f1d28-d4d4-4e0f-ae32-83960160f544.png align="center")

gradle -&gt; libs.versions.toml 파일을 추가한다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695730955676/daa99915-f414-44b0-97dd-aaa2cb1f5b5b.png align="center")

내용은 이런 식으로 들어간다. 전체 코드는 [여기](https://github.com/Akatsuki-USW/Buzzzzing-Android/blob/develop/gradle/libs.versions.toml)에서 확인 가능하다.

코드를 살펴보면 \[versions\], \[libraries\], \[bundles\], \[plugins\] 키워드가 있다.

* versions : 라이브러리 버전
    
* libraries : 라이브러리 의존성
    
* bundles : 라이브러리를 묶어서 한 번에 선언
    
* plugins: 어떤 플러그인을 사용하는지
    

이제 버전 카탈로그를 사용하기 위한 준비는 끝났다.

### 2\. 디펜던시 이전

Kotlin DSL로 작성된 Gradle에 Version Catalog를 적용해보자. (Gradle를 Groovy를 사용하고 있다면 Kotlin DSL로 이동하는 방법은 [여기](https://kotlinworld.com/121)에서 확인할 수 있다.)

```kotlin
implementation("com.google.android.gms:play-services-auth:20.5.0")
```

예를 들어 위 디펜던시를 추가하고 싶다면 libs.versions.toml에 다음과 같이 입력하면 된다.

```kotlin
[versions]
play-services-auth = "20.5.0"

[libraries]
google-services-auth = { group = "com.google.android.gms", name = "play-services-auth", version.ref = "play-services-auth" }
```

libraries는 `:` 을 기준으로 group, name, version.ref으로 나뉜다. version.ref에는 \[versions\]에 적어놓은 이름을 적어주면 된다.

그 이후 Sync를 하고 build.gradle에 아래처럼(`implementation(libs.google.services.auth)`) 추가해주면 된다.

```kotlin
implementation(libs.google.services.auth) // 버전 카탈로그 적용 후
```

아까 적었던 `-` 은 `.` 으로 대체할 수 있다. IDE 상에서 입력해보면, catalog 별로 자동완성을 지원함을 알 수 있다. 따라서 간편하게 추가가 가능하다.

### 3\. 플러그인 이전

기존 플러그인이 아래와 같다면

```kotlin
// Top-level `build.gradle.kts` file
plugins {
   id("com.android.application") version "7.4.1" apply false
}

// Module-level `build.gradle.kts` file
plugins {
   id("com.android.application")
}
```

버전 카탈로그에 다음과 같이 적어주고 ...

```kotlin
[versions]
android-gradle-plugin = "7.4.1"

[plugins]
android-application = { id = "com.android.application", version.ref = "android-gradle-plugin" }
```

아래의 코드로 바꿔주면 된다.

```kotlin
// Top-level build.gradle.kts
plugins {
   alias(libs.plugins.android.application) apply false
}

// module build.gradle.kts
plugins {
   alias(libs.plugins.android.application)
}
```

### 4\. Bundle 사용법

예를 들어 버전 카탈로그가 아래와 같다면 ...

```kotlin
[versions]
retrofit = "2.6.2"

[libraries]
# Network
retrofit-core = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }
retrofit-adapter-rxjava = { module = "com.squareup.retrofit2:adapter-rxjava2", version.ref = "retrofit" }
retrofit-converter-gson = { module = "com.squareup.retrofit2:converter-gson", version.ref = "retrofit" }

[bundles]
retrofit = [
    "retrofit-core",
    "retrofit-adapter-rxjava",
    "retrofit-converter-gson"
]
```

번들을 사용하여 `retrofit-core`, `retrofit-adapter-rxjava`, `retrofit-converter-gson` 를 한번에 추가할 수 있다.

```kotlin
//build.gradle.kts

dependencies {
  implementation(libs.bundles.retrofit) // retrofit(core, rxjava, converter)
}
```

## Convention Plugin 적용 순서

version catalog로 이전이 끝났다면 이제 Convention Plugin을 적용할 수 있다.

### 1\. build-logic 모듈 생성

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695814322312/01ce1abe-1514-4e83-807e-5b9038fe867e.png align="center")

모듈 생성 후 안의 내용은 전부 비워주고 `settings.gradle.kts` 을 추가한다.

```kotlin
//settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}

rootProject.name = "build-logic"
include(":convention")
```

그리고 build-logic안에 convention이라는 모듈을 추가한다. (Java or Kotlin Library로 만들자.)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695814725961/b05a4bbe-e699-4a00-b1f4-f2b3c58a63b4.png align="center")

이런 패키지 구조를 가지면 된다.

그리고 :build-logic:convention의 build.gradle를 다음과 같이 수정한다.

```kotlin
@Suppress("DSL_SCOPE_VIOLATION") // TODO: Remove once KTIJ-19369 is fixed
plugins {
    `kotlin-dsl`
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

dependencies {
    compileOnly(libs.android.gradle.plugin)
    compileOnly(libs.kotlin.gradle.plugin)
    compileOnly(libs.ksp.gradle.plugin)
}
```

root 수준의 settings.gradle.kts는 이렇게 수정한다. (자동으로 추가된 `include(":build-logic")`, `include(":build-logic:convention")`를 지워줘야한다.)

```kotlin
pluginManagement {
    includeBuild("build-logic")
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

// ...
include(":app")
// include(":build-logic") // remove
// include(":build-logic:convention") // remove
```

이제 KotlinAndroid, Compose 세팅을 도와주는 확장 함수를 build-logic/convention 안에 추가한다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695815934634/143388b2-1f60-439c-b649-5824dd51b8d9.png align="center")

```kotlin
// KotlinAndroid.kt
internal fun Project.configureKotlinAndroid(
    commonExtension: CommonExtension<*, *, *, *, *>,
) {
    commonExtension.apply {
        compileSdk = 34

        defaultConfig {
            minSdk = 26

            testInstrumentationRunner = "android.support.test.runner.AndroidJUnitRunner"
            vectorDrawables.useSupportLibrary = true
        }

        compileOptions {
            sourceCompatibility = JavaVersion.VERSION_17
            targetCompatibility = JavaVersion.VERSION_17
        }

        kotlinOptions {
            jvmTarget = JavaVersion.VERSION_17.toString()
        }
    }
}

fun CommonExtension<*, *, *, *, *>.kotlinOptions(block: KotlinJvmOptions.() -> Unit) {
    (this as ExtensionAware).extensions.configure("kotlinOptions", block)
}
```

```kotlin
// AndroidCompose.kt
internal fun Project.configureAndroidCompose(
    commonExtension: CommonExtension<*, *, *, *, *>,
) {
    val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")

    commonExtension.apply {
        buildFeatures.compose = true

        composeOptions {
            kotlinCompilerExtensionVersion = libs.findVersion("compose.compiler").get().requiredVersion
        }
    }

    dependencies {
        // Disabling to work with Alpha
        "api"(platform(libs.findLibrary("compose.bom").get()))
        "implementation"(libs.findBundle("compose").get())
        "debugImplementation"(libs.findBundle("compose.debug").get())
    }
}
```

### 2\. 컨벤션 플러그인 추가

convention에 app 모듈의 build.gradle 설정 플러그인을 추가한다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695816232242/119220d1-1243-4cae-b755-061be7978b78.png align="center")

```kotlin
internal class AndroidApplicationConventionPlugin : Plugin<Project> {

    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.application")
                apply("org.jetbrains.kotlin.android")
            }

            extensions.configure<ApplicationExtension> {
                configureKotlinAndroid(this)
            }
        }
    }
}
```

```kotlin
class AndroidApplicationComposeConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply("com.android.application")
            val extension = extensions.getByType<BaseAppModuleExtension>()
            configureAndroidCompose(extension)
        }
    }
}
```

이어서 다른 컨벤션들도 추가해준다. 코드는 [여기](https://github.com/Akatsuki-USW/Buzzzzing-Android)서 확인할 수 있다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695816571901/75f772c9-c287-41b7-9cbc-99fd059b9b6f.png align="center")

프로젝트에 맞게 추가하면 된다. 나는 7개의 컨벤션을 만들었다.

이제 마지막으로 build-logic:convention 모듈의 build.gradle.kts 파일에 컨벤션을 등록해주면 된다.

```kotlin
@Suppress("DSL_SCOPE_VIOLATION") // TODO: Remove once KTIJ-19369 is fixed
plugins {
    `kotlin-dsl`
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

dependencies {
    compileOnly(libs.android.gradle.plugin)
    compileOnly(libs.kotlin.gradle.plugin)
    compileOnly(libs.ksp.gradle.plugin)
}

gradlePlugin {
    plugins {
        register("AndroidApplicationPlugin") {
            id = "done.plugin.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
        register("AndroidApplicationComposePlugin") {
            id = "done.plugin.application.compose"
            implementationClass = "AndroidApplicationComposeConventionPlugin"
        }
        // ...
    }
}
```

id는 컨벤션 플러그인의 이름이고 (직접 지정하면 된다.) implementationClass는 id에 매칭되는 클래스 이름이다.

### 3\. 적용해보기

예를 들어서 A 모듈에 Hilt를 추가하고 싶다면 귀찮은 설정 없이 A 모듈의 build.gradle.kts plugin에 `id("buzzzzing.plugin.hilt")` 한줄만 추가하면 된다.

```kotlin
plugins {
    id("done.plugin.hilt")
}
```

feature를 다수의 모듈로 관리하게 되면 build.gradle에 중복 코드가 많아지는데 위에서 언급한 `FeatureConventionPlugin`을 사용하면 쉽게 build.gradle를 관리할 수 있다.

블로그에 적어놓은 내용이 반영된 깃헙 주소를 아래 첨부한다.

[https://github.com/Akatsuki-USW/Buzzzzing-Android](https://github.com/Akatsuki-USW/Buzzzzing-Android)

## 참고

[https://brunch.co.kr/@purpledev/46](https://brunch.co.kr/@purpledev/46)

[https://medium.com/prnd/%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EB%B2%84%EC%A0%84%EA%B4%80%EB%A6%AC-%EB%8D%94%EC%9D%B4%EC%83%81-buildsrc%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%88%EC%84%B8%EC%9A%94-feat-catalog-%ED%97%A4%EC%9D%B4%EB%94%9C%EB%9F%AC-%EA%B8%B0%EC%88%A0%EB%B8%94%EB%A1%9C%EA%B7%B8-710b4ca0870d](https://medium.com/prnd/%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EB%B2%84%EC%A0%84%EA%B4%80%EB%A6%AC-%EB%8D%94%EC%9D%B4%EC%83%81-buildsrc%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%88%EC%84%B8%EC%9A%94-feat-catalog-%ED%97%A4%EC%9D%B4%EB%94%9C%EB%9F%AC-%EA%B8%B0%EC%88%A0%EB%B8%94%EB%A1%9C%EA%B7%B8-710b4ca0870d)

[https://developer.android.com/build/migrate-to-catalogs#kts](https://developer.android.com/build/migrate-to-catalogs#kts)

[https://medium.com/@kingjinho/android-gradle-version-catalog-%EC%A0%81%EC%9A%A9-85f41ae2ef6e](https://medium.com/@kingjinho/android-gradle-version-catalog-%EC%A0%81%EC%9A%A9-85f41ae2ef6e)