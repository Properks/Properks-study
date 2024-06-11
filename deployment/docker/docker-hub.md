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
   + `{version}`에 필요한 버전은 오류에서 `linux/amd64/v3`처럼 보여준다.
   
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

1. EC2 update and upgrade

   ```shell
   sudo apt update
   ```

   ```shell
   sudo apt upgrade
   ```
   
2. EC2에 Docker 설치

   ```shell
   sudo apt install docker.io
   ```
3. Permission denied error 해결

   ```shell
   sudo chmod 666 /var/run/docker.sock
   ```
   `/var/run/docker.sock`의 권한을 소유자, 그룹, 사용자가 읽기 쓰기를 할 수 있도록 바꾸어 준다.


### 4. Docker hub에서 image를 pull하고 실행

1. EC2에 image pull 하기

   ```shell
   docker pull {docker username}/{image name}:{tag}
   ```

   <details>
   <summary>Example</summary>

   ```shell
   docker pull user/image:tag
   ```
   </details> 


2. EC2에서 image 실행

   2-1. 환경 변수 없이 EC2 실행
   ```shell
   docker run -rm -d -p 8080:8080 {docker username}/{image name}:{tag}
   ```

   <details>
   <summary>Example</summary>

   ```shell
   docker run -rm -d -p 8080:8080 user/image:tag
   ```
   </details> 

   2-2. 환경 변수 추가하여 EC2 실행

   ```shell
   docker run -e DB_URL=DB_URL -e DB_USERNAME=DB_USERNAME -e DB_PASSWORD=DB_PASSWORD -rm -d -p 8080:8080 {docker username}/{image name}:{tag}
   ```

   <details>
   <summary>Example</summary>

   ```shell
   docker run -e DB_URL=jdbc:mysql://localhost:3306/schema -e DB_USERNAME=root -e DB_PASSWORD=1234 -rm -d -p 8080:8080 {docker username}/{image name}:{tag}
   ```
   </details> 
   
   + 환경 변수가 여러 개일 경우 -e를 앞에 붙이면서 추가해준다.
   + 해당 build하는 과정에서 permission denied error가 발생할 수 있다.
     + error 발생하는 경우 [해당 경로](#3-ec2에-도커-설치-및-환경-설정)의 3번 문항으로 이동하여 명령어를 실행하면 된다.
