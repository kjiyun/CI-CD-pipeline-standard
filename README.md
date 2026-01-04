## CI/CD Pipeline Standard

## CI 적용 방법
```Yaml
name: Spring Boot & Gradle CI

on:
  pull_request:
    branches: [ "main" ] # CI를 실행할 대상 브랜치

jobs:
  ci:
    uses: kjiyun/CI-CD-pipeline-standard/.github/workflows/java-gradle-ci.yml@master
    with:
      java-version: "17"           # 필수
      distribution: "temurin"      # 기본값: temurin
      build-command: false         # 기본값: './gradlew build --no-daemon assemble'
      test-command: false          # 기본값: './gradlew test'
      upload-artifacts: true       # 기본값: true
      write-application-yml: true  # 기본값: false
    secrets:
      APPLICATION: ${{ secrets.APPLICATION }}
```
### 필수 설정 항목
- `on.pull_request.branches`: CI를 실행할 대상 브랜치를 명시합니다.
- `jobs.ci.uses`: 공통 CI 워크플로우를 참조하는 부분으로, 고정 값입니다.
- `java-version`: 사용할 JDK 버전을 반드시 지정해야 합니다.

### 선택 설정 항목
- `distribution`: JDK 배포판을 지정합니다.
  <br>작성하지 않으면 기본값으로 temurin이 사용됩니다.
- `build-command`: 빌드 명령어를 커스터마이징할 때 사용합니다.
  <br>지정하지 않으면 기본값인 ./gradlew build --no-daemon assemble가 실행됩니다.
- `test-command`: 테스트 명령어를 별도로 지정할 수 있습니다.
  <br>미작성 시 기본 테스트 명령어가 실행됩니다.
- `upload-artifacts`: 빌드 결과물(JAR 등)을 아티팩트로 업로드할지 여부를 설정합니다.
  <br>기본값은 true입니다.

### application.yml 사용 시 설정 방법
- `application.yml`을 **GitHub Secret**으로 관리하는 경우 `write-application-yml: true`로 설정합니다.
- 이후 `secrets` 섹션에서 `APPLICATION: ${{ secrets.(저장한 Secret 이름) }}` 형태로 매핑합니다.

## CD 적용 방법
```Yaml
name: Spring Boot & Gradle CD

on:
  push:
    branches: [ "main" ]  # CD를 실행할 대상 브랜치

jobs:
  cd:
    uses: kjiyun/CI-CD-pipeline-standard/.github/workflows/java-gradle-cd.yml@master
    with:
      java-version: "17"              # 필수
      distribution: "temurin"         # 기본값: temurin
      image-name: kahlua-web          # 필수 (Docker 이미지 이름)
      write-application-yml: true     # 기본값: false
    secrets:
      APPLICATION: ${{ secrets.APPLICATION }}
      DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
      DOCKER_REPO: ${{ secrets.DOCKER_REPO }}
      HOST_ID: ${{ secrets.HOST_ID }}
      USERNAME: ${{ secrets.USERNAME }}
      PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
```
### 필수 설정 항목
- `on.push.branches`: CD를 실행할 대상 브랜치를 명시합니다.
- `jobs.cd.uses`: 공통 CD 워크플로우를 참조하는 부분으로, 고정 값입니다.
- `java-version`: 배포에 사용할 JDK 버전을 반드시 지정해야 합니다.
- `image-name`: Docker 이미지 이름을 반드시 지정해야 합니다.
- `secrets`: CD 단계에서 사용되는 모든 Secret은 필수입니다.
  <br>각 값은 해당 레포의 GitHub Secrets에 정확한 이름으로 등록되어 있어야 합니다.

### 선택 설정 항목
- `distribution`: JDK 배포판을 지정합니다.
  <br>작성하지 않으면 기본값으로 temurin이 사용됩니다.
- `write-application-yml`
  `application.yml`을 **GitHub Secret**으로 관리하는 경우 `true`로 설정합니다.
  <br>설정 시 CI/CD 실행 중 `application.yml 파일이 자동 생성됩니다.

### Secret 설정 주의사항
- `secrets` 하위에 명시된 이름은
  <br>**Repository Settings → Secrets and variables → Actions**에 등록된 Secret 이름과 정확히 일치해야 합니다.