# 🐳 도커 & 쿠버네티스 2주차 강의노트
 #강의노트 #Docker #Github_Action

# 🚀 Github Action으로 CI/CD 구축하기

## Github Action이란?

Github에서 제공하는 CI/CD 플랫폼으로, Pull Request, Commit, Issue 생성 등 깃허브 레포지토리의 여러가지 이벤트들이 발생할 때 자동으로 빌드, 테스트, 배포를 할 수 있도록 해 주는 서비스이다.

### 구성요소

**Workflows**
하나 이상의 Job으로 구성된 자동화된 프로세스.

**Events**
워크플로우가 시작되는 트리거이다. Pull Request, Commit, Issue 생성 등이 이벤트가 될 수 있다.

**Jobs**
하나의 Runner에서 실행되는 여러 단계의 작업들이다. 각 단계를 Step이라고 하고 각 Step은 Action 또는 Shell Script가 될 수 있다.

**Runners**
Workflow를 실행하는 서버이다. 깃허브에서 자체적으로 Windows, MacOS, ubuntu등 여러가지 서버를 제공해준다. 자신의 서버를 대신 사용할수도 있다.

**Actions**
자주 사용하는 작업들을 한번에 수행해주는 단위이다. Job의 한 단계가 될 수 있다.

## 샘플 프로젝트 fork하기

https://github.com/dockersamples/example-voting-app

fork한 후 워크플로우 탭에 들어가 워크플로우를 활성화시켜주어야 한다.

## Github Action 생성하기

깃허브 레포지토리의 `.github/workflows` 디렉토리에 YAML 파일을 생성하여 워크플로우를 생성한다.

**learn-github-actions.yaml**
```yaml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

이제 github에 커밋된 사항을 푸시하면 자동으로 워크플로우가 실행된다.

실행된 워크플로우는 **Action** 탭에서 확인할 수 있다.

## EC2 생성 및 SSH 접속

1. EC2 인스턴스 생성 시 반드시 키페어를 선택해주고, 개인 키를 안전한 곳에 보관한다.
2. 네트워크 인바운드 규칙에서 SSH를 반드시 오픈해 주어야 한다.
3. 추가로 5000, 5001번 포트도 열어준다.

**Unix & MacOS**
우선 개인 키의 권한을 수정해준다.
```bash
chmod 400 /path/to/keypair 
```

그리고 연결을 시도한다.
```bash
ssh -i /path/to/keypair ubuntu@<ec2-instance-public-ip>
```

**Windows**
아래 글을 참고하여 접속
[윈도우 터미널을 이용해 EC2 접속하기 \(5\)](https://wookim789.tistory.com/34)

## EC2에 Docker 설치 및 Docker Compose로 서비스 시작

이전 강의자료 참고

## 레포지토리에 Action Secret 생성하기

[Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
[Github\) Github actions에서 Secrets로 환경변수 관리하기](https://velog.io/@2ast/Github-Github-actions%EC%97%90%EC%84%9C-Secrets%EB%A1%9C-%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0)

fork한 레포지토리에 `keypair.pem`의 컨텐츠, 호스트 주소(ec2 퍼블릭 IP), 사용자 이름(ubuntu)를 각각  `SSH_PRIVATE_KEY`, `SSH_HOST`, `USER_NAME`으로 생성해준다.

## 자동배포 워크플로우

**deployment.yaml**
```yaml
name: Deploy
run-name: Auto deployment to ec2
on:
  push:
    branches: [ main ]
jobs:
  Deploy:
    name: Deploy to EC2
    runs-on: ubuntu-latest
    steps:
      - name: Pull & Deploy
        env:
          PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          HOSTNAME: ${{ secrets.SSH_HOST }}
          USER_NAME: ${{ secrets.USER_NAME }}
        run: |
          echo "$PRIVATE_KEY" > private_key && chmod 400 private_key
          ssh -o StrictHostKeyChecking=no -i private_key ${USER_NAME}@${HOSTNAME} '
            cd /home/ubuntu/example-voting-app
            sudo git checkout main
            sudo git pull origin main
            sudo docker compose up -d
            '
```

이후 수정 후 깃허브에 푸시하면 자동으로 배포가 진행된다.

# 👾 Docker 컨테이너 디버깅

### 실행중인 컨테이너에 접속하기

```bash
docker exec -it <container-id> bash
```

### 컨테이너 로그 확인하기

```bash
docker container logs <container-id>
```

