# n8n Self-Hosting 가이드

n8n을 직접 호스팅하는 방법에 대한 종합 가이드입니다. 비용 효율적이고 자유로운 워크플로우 관리를 위한 다양한 설치 옵션을 알아봅니다.

## 목차

- [셀프호스팅 방식 개요](#셀프호스팅-방식-개요)
- [로컬 컴퓨터에 설치하기](#로컬-컴퓨터에-설치하기)
  - [Docker로 설치하기](#docker로-설치하기)
  - [로컬 설치의 장단점](#로컬-설치의-장단점)
- [웹훅 연결 설정하기](#웹훅-연결-설정하기)
  - [임시 URL로 웹훅 연결하기](#임시-url로-웹훅-연결하기)
  - [고정 커스텀 도메인 연결하기](#고정-커스텀-도메인-연결하기)
- [클라우드 서버에 설치하기](#클라우드-서버에-설치하기)
  - [Railway를 이용한 설치 방법](#railway를-이용한-설치-방법)
  - [클라우드 설치의 장단점](#클라우드-설치의-장단점)

## 셀프호스팅 방식 개요

n8n(n eight n)을 셀프호스팅하는 방법은 크게 두 가지로 나눌 수 있습니다:

1. **내 컴퓨터에 설치하는 방식**
   - npm을 활용한 로컬 디렉토리 설치
   - Docker를 활용한 컨테이너 설치

2. **클라우드 서버에 설치하는 방식**
   - GCP, AWS 같은 클라우드 서비스 활용
   - Railway, Digital Ocean과 같은 PaaS 서비스 활용

이 가이드에서는 설치와 운영의 간편함, 가성비 등을 고려하여 **로컬에 Docker로 설치하는 방법**과 **Railway로 n8n을 셀프호스팅하는 방법**에 대해 중점적으로 알아보겠습니다.

## 로컬 컴퓨터에 설치하기

### Docker로 설치하기

#### 1. Docker Desktop 설치

먼저 [Docker 공식 웹사이트](https://www.docker.com/)에서 Docker Desktop을 다운로드하여 설치합니다.

#### 2. n8n 이미지 다운로드

Docker Desktop에서 검색창에 "n8n"을 검색하고, 가장 많이 다운로드된 이미지를 Pull 합니다.

#### 3. n8n 컨테이너 실행

이미지를 다운로드한 후 다음과 같이 설정하여 실행합니다:

- **컨테이너 이름**: 원하는 이름 지정 (예: n8n-docker)
- **포트**: 5678
- **호스트 경로**: 로컬 컴퓨터에 n8n 관련 데이터를 저장할 경로 지정 (예: n8n-data 폴더)
- **컨테이너 경로**: `/home/node/.n8n`

Docker Desktop의 UI를 통해 설정하거나, 다음 명령어로 실행할 수 있습니다:

```bash
# 맥/리눅스
docker run -rm \
  --name n8n-docker \
  -p 5678:5678 \
  -v /path/to/your/n8n-data:/home/node/.n8n \
  n8nio/n8n:latest
```

```powershell
# 윈도우
docker run -rm `
  --name n8n-docker `
  -p 5678:5678 `
  -v C:\path\to\your\n8n-data:/home/node/.n8n `
  n8nio/n8n:latest
```

이후 브라우저에서 `http://localhost:5678`으로 접속하여 이메일과 비밀번호를 설정하면 n8n을 사용할 수 있습니다.

### 로컬 설치의 장단점

#### 장점
- 워크플로우 개수 제한 없음
- 모든 기능 무료로 사용 가능
- 자체 데이터 저장 및 관리 가능

#### 단점
- 컴퓨터와 Docker 컨테이너가 실행 중일 때만 작동
- 컴퓨터가 꺼져 있을 때는 자동화 워크플로우가 실행되지 않음
- 기본적으로 웹훅 URL이 로컬호스트로 제한됨

## 웹훅 연결 설정하기

로컬 설치 시 외부에서 웹훅을 받기 위해서는 내부 포트를 외부 주소와 연결해야 합니다.

### 임시 URL로 웹훅 연결하기

#### 1. Cloudflare 설치

**맥:**
```bash
brew install cloudflared
```

**윈도우:**
```powershell
winget install --id Cloudflare.cloudflared
```

#### 2. 임시 터널 생성

```bash
cloudflared tunnel --url http://localhost:5678
```

이 명령어를 실행하면 임시 URL이 생성됩니다. 이 URL은 터미널을 종료하거나 컴퓨터를 재시작하면 사라집니다.

#### 3. 웹훅 URL 설정으로 Docker 컨테이너 재실행

기존 컨테이너를 삭제하고 새로운 설정으로 다시 실행합니다:

```bash
# 맥/리눅스
docker run -rm \
  --name n8n-docker \
  -p 5678:5678 \
  -v /path/to/your/n8n-data:/home/node/.n8n \
  -e WEBHOOK_URL=<임시_URL> \
  n8nio/n8n:latest
```

```powershell
# 윈도우
docker run -rm `
  --name n8n-docker `
  -p 5678:5678 `
  -v C:\path\to\your\n8n-data:/home/node/.n8n `
  -e WEBHOOK_URL=<임시_URL> `
  n8nio/n8n:latest
```

### 고정 커스텀 도메인 연결하기

웹훅을 지속적으로 사용할 경우, 커스텀 도메인을 구매하여 연결하는 것이 효율적입니다.

#### 1. 도메인 구매 및 Cloudflare 연결

1. GoDaddy, Namecheap, 가비아 등에서 도메인을 구매합니다.
2. [Cloudflare 웹사이트](https://www.cloudflare.com/)에 로그인하고 도메인을 추가합니다.
3. Cloudflare에서 제공하는 nameserver를 도메인 제공업체의 설정에 적용합니다.
4. Cloudflare에서 인증 요청을 진행하고 활성화될 때까지 기다립니다.

#### 2. Cloudflare 터널 설정

**로그인 및 인증:**
```bash
cloudflared login
```

**터널 생성:**
```bash
cloudflared tunnel create n8n-tunnel
```

**config.yml 파일 생성:**

**맥:**
```bash
cat <<EOF > ~/.cloudflared/config.yml
tunnel: n8n-tunnel
credentials-file: /Users/username/.cloudflared/your-credential-file.json

ingress:
  - hostname: n8n.yourdomain.com
    service: http://localhost:5678
  - service: http_status:404
EOF
```

**윈도우:**
```powershell
notepad $env:USERPROFILE\.cloudflared\config.yml
```
메모장에 다음 내용을 입력하고 저장:
```yaml
tunnel: n8n-tunnel
credentials-file: C:\Users\YourUsername\.cloudflared\your-credential-file.json

ingress:
  - hostname: n8n.yourdomain.com
    service: http://localhost:5678
  - service: http_status:404
```

**도메인과 터널 연결:**
```bash
cloudflared tunnel route dns n8n-tunnel n8n.yourdomain.com
```

#### 3. Docker 컨테이너 설정 및 실행

```bash
# 맥/리눅스
docker run -d \
  --name n8n-docker \
  --restart unless-stopped \
  -p 5678:5678 \
  -v /path/to/your/n8n-data:/home/node/.n8n \
  -e WEBHOOK_URL=https://n8n.yourdomain.com \
  n8nio/n8n:latest
```

```powershell
# 윈도우
docker run -d `
  --name n8n-docker `
  --restart unless-stopped `
  -p 5678:5678 `
  -v C:\path\to\your\n8n-data:/home/node/.n8n `
  -e WEBHOOK_URL=https://n8n.yourdomain.com `
  n8nio/n8n:latest
```

**n8n 실행 후 터널 실행:**
```bash
cloudflared tunnel run n8n-tunnel
```

**참고:** 컴퓨터를 재시작하면 Docker는 자동으로 실행되도록 설정할 수 있지만, 터널은 수동으로 다시 실행해야 합니다:
```bash
cloudflared tunnel run n8n-tunnel
```

## 클라우드 서버에 설치하기

24시간 실행이 필요하거나 컴퓨터를 계속 켜두기 어려운 경우, 클라우드 서버에 설치하는 것이 좋습니다.

### Railway를 이용한 설치 방법

[Railway](https://railway.app/)는 서버 설정과 관리를 간소화해주는 PaaS 서비스입니다.

#### 1. Railway 가입 및 계획 선택

1. Railway 웹사이트에 가입합니다.
2. 최소 $5/월 요금제를 선택합니다. (사용량에 따라 추가 비용이 발생할 수 있습니다)

#### 2. n8n 프로젝트 생성

1. 새 프로젝트를 생성합니다.
2. 템플릿에서 "n8n"을 검색합니다.
3. "Webhook Processor" 템플릿을 선택합니다.

이 템플릿은 Queue Mode로 설정되어 다음 컴포넌트를 함께 배포합니다:
- **PostgreSQL**: 데이터 저장
- **Redis**: 작업 분배
- **Primary**: 메인 n8n 인스턴스
- **Worker**: 자동화 워크플로우 실행
- **Webhook Processor**: 웹훅 빠른 처리

#### 3. n8n 접속

배포가 완료되면 Primary 서비스의 Settings에서 생성된 URL 링크로 접속할 수 있습니다.

#### 버전 업그레이드

버전 업그레이드가 필요할 때는 Railway의 Deployments 섹션에서 Primary, Worker, Webhook Processor 3개 서비스를 선택하여 Redeploy를 실행합니다.

### 클라우드 설치의 장단점

#### 장점
- 24시간 자동화 워크플로우 실행 가능
- 웹훅 설정이 기본으로 포함되어 있음
- 컴퓨터 상태와 관계없이 실행됨

#### 단점
- 월 $5부터 시작하는 비용 발생 (사용량에 따라 증가 가능)
- 클라우드 서비스에 의존

## 결론

n8n 셀프호스팅 방식은 사용 목적과 상황에 맞게 선택하는 것이 중요합니다:

- **테스트 용도나 간헐적 사용**: 로컬 Docker 설치
- **상시 자동화가 필요한 경우**: Railway와 같은 클라우드 서비스 활용

두 방식 모두 n8n 공식 서비스보다 저렴하면서도 무제한에 가까운 워크플로우 사용이 가능하다는 장점이 있습니다.