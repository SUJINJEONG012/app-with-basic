# app-with-argocd

## 결과 
1. 애플리케이션을 빌드 -> jar생성.
2. Docker 이미지를 빌드
3. Docker Hub에 이미지 푸시
4. 멀티아키텍처 지원(ARM, AMD)
5. 이후, ArgoCD를 통해 Docker Hun에서 이미지를 가져와 쿠버네티스 클러스터에 배포

push 이벤트 발생 시(main 브랜치에 새 코드가 푸시될 때) 이 워크플로가 실행
runs-on: ubuntu-latest: 최신 Ubuntu 환경에서 실행.
actions/checkout@v4 : GitHub 저장소의 코드를 가져옴

```
cd ./src/main/resources
touch ./application.properties
echo "${{ secrets.PROPERTIES }}" > ./application.properties
```
application.properties 파일을 생성하고, GitHub Secrets에서 제공된 PROPERTIES 값을 파일에 작성

Maven을 사용하여 애플리케이션을 빌드
```
mvn -B -DskipTests package --file pom.xml
```

빌드된 JAR 파일을 app.jar로 이름을 변경
```
mv ./target/*.jar ./target/app.jar
```

actions/upload-artifact@v4
app.jar 파일을 GitHub Actions의 아티팩트로 업로드하여 저장
```
- uses: actions/upload-artifact@v4
  with:
    name: app
    path: ./target/*.jar
```

Docker Hub에 로그인,로그인 정보는 GitHub Secrets에서 가져옴
```
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

QEMU를 설정하여 여러 아키텍처의 Docker 이미지를 빌드할 수 있도록 준비
```
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3
```

Docker Buildx를 설정하여 멀티 플랫폼 이미지를 빌드
```
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```

Docker 이미지를 빌드하고 Docker Hub로 푸시
```
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
platforms: linux/arm64,linux/amd64
    tags: ${{ secrets.DOCKERHUB_USERNAME }}/${{ secrets.DOCKERHUB_REPOSITORY }}:${{ github.sha }}
```
