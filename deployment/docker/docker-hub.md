# Docker hub를 사용한 Spring boot project 배포

## 준비 사항

- Docker ID
- EC2 Instance
- 배포할 Spring boot 프로젝트

## 절차

### 1. Spring boot project 내의 환경 설정
1. bootJar 실행
    ```shell
   ./gradlew bootJar
   ```
   
   1 - 1.  (선택) build.gradle 설정하여 jar 파일 이름 설정
    ```
   bootJar {
	    archiveFileName = '{filename}.jar'
    }
   ``` 
2. Dockerfile 작성

   ```dockerfile
   FROM openjdk:17

   ARG JAR_FILE=/build/libs/{filename}.jar

   COPY ${JAR_FILE} app.jar

   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```
   2-1. 환경 변수를 설정한 경우 (예시: MySQL 환경변수)
   ```dockerfile
   FROM openjdk:17

   ARG JAR_FILE=/build/libs/{filename}.jar

   ARG DB_URL
   ARG DB_USERNAME
   ARG DB_PASSWORD

   ENV DB_URL=${DB_URL}
   ENV DB_USERNAME=${DB_USERNAME}
   ENV DB_PASSWORD=${DB_PASSWORD}

   COPY ${JAR_FILE} app.jar

   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```

### 2. Docker hub에 image를 push
1. Image 생성

   ```shell
   docker build -t {docker username}/{image name}:{tag} .
   ```
   
   2-1. 만약 AWS instance에서 image build 시 버전 오류 발생 시

   ```shell
   docker build --platform {version} -t {docker username}/{image name}:{tag} .     
   ```

   + tag는 생략 가능, 생략 시 자동으로 latest가 tag로 붙는다.
   + 2-1에서의 필요한 버전은 오류와 같이 나오는데 이를 `--platform`뒤에 넣으면 된다.
   
   <details>
   <summary>Example</summary>

   ```shell
   docker build -t user/image:tag .
   ```

   ```shell
   docker build --platform linux/amd64/v3 -t user/image:tag .     
   ``` 
   </details>

   
2. Image push

   ```shell
   docker push {docker username}/{image name}:{tag}
   ```
   <details>
   <summary>Example</summary>

   ```shell
   docker push user/image:tag
   ```
   </details> 
   

### 3. EC2에 도커 설치 및 환경 설정
### 4. Docker hub에서 image를 pull하고 실행