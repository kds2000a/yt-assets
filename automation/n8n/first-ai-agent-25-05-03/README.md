# n8n AI Agent 생성 가이드

## 목차
1. [AI Agent 개요](#ai-agent-개요)
2. [n8n AI Agent 기본 설정](#n8n-ai-agent-기본-설정)
3. [Google 서비스 연동 설정](#google-서비스-연동-설정)
4. [AI Agent 툴 설정](#ai-agent-툴-설정)
5. [시스템 프롬프트 설정](#시스템-프롬프트-설정)
6. [타임존 설정](#타임존-설정)
7. [테스트 및 배포](#테스트-및-배포)

## AI Agent 개요

![n8n-aiagent-example](./n8n-ai-agent.png)

n8n에서 AI를 활용하는 방법은 크게 두 가지가 있습니다:

1. **AI를 포함한 자동화 워크플로우(AI Chain)**: 미리 정해진 순서대로 실행되며, 그 중 일부 단계에서 AI 기능을 활용
2. **AI Agent**: AI가 직접 워크플로우 순서를 판단하고, 필요한 툴을 활용해 결과물 생성

**AI Agent의 장점**:
- 설정 방법이 간단함 (툴만 연결하면 됨)
- 다양한 변수에 유연하게 대응 가능
- 복잡한 워크플로우 설계 불필요

**AI Agent의 단점**:
- AI 판단에 따라 결과가 달라질 수 있음
- 프롬프트가 명확하지 않거나 복잡한 경우 원치 않는 결과 발생 가능

## n8n AI Agent 기본 설정

![n8n-aiagent-example](./ai-agent-workflow-ex.png)

### 1. 트리거 설정

1. 새 워크플로우 생성
2. 트리거로 `Chat message received` 선택

### 2. AI Agent 노드 추가

1. 우측 + 버튼 클릭
2. AI 카테고리에서 `AI Agent` 선택

### 3. AI 모델 설정 (Chat Model)

AI Agent의 "두뇌" 역할을 하는 모델을 선택합니다.

1. OpenAI API 키 준비 (없을 경우 platform.openai.com에서 발급)
   - 참고: OpenAI 데이터 공유에 동의하면 일정량의 토큰을 무료로 제공받을 수 있음
2. n8n에서 Credential 추가 (Add credential 클릭)
3. API Key 입력 후 인증
4. 모델 선택 (예: gpt-4.1 mini)

### 4. 메모리 설정

AI Agent가 대화 맥락을 기억할 수 있도록 메모리 설정:

1. Memory 탭에서 `Simple Memory` 선택
2. 기억할 메시지 수 설정 (예: 10개)

## Google 서비스 연동 설정

n8n에서 Google 서비스를 연동하기 위한 설정 방법:

### 1. Google Cloud Console 설정

1. [console.cloud.google.com](https://console.cloud.google.com) 접속
2. 프로젝트 생성
3. API 및 서비스 메뉴로 이동
4. 사용할 API 활성화 (Google Calendar, Google Sheets, Gmail, Google Drive 등)

### 2. OAuth 동의 화면 설정

1. OAuth 동의 화면 메뉴로 이동
2. 앱 정보 입력
3. 테스트 사용자에 본인 이메일 추가

### 3. 사용자 인증 정보 설정

1. 사용자 인증 정보 메뉴로 이동
2. 사용자 인증 정보 만들기 > OAuth 클라이언트 ID 선택
3. 웹 애플리케이션 선택
4. n8n의 리디렉션 URI 입력 (n8n에서 표시되는 URI 사용)
5. 생성 후 클라이언트 ID와 보안 비밀번호(시크릿) 저장

### 4. 데이터 액세스 범위 설정

1. 범위 추가 또는 삭제 메뉴로 이동
2. 필요한 서비스의 범위 추가 (Calendar, Sheets, Gmail, Drive 등)
3. 저장

## AI Agent 툴 설정

### 1. Google Calendar 툴 설정

일정 조회 도구:
1. 도구 추가 > Google Calendar > Event > Get Many 선택
2. 인증 정보 연결 (위에서 설정한 Google OAuth 정보)
3. 캘린더 선택
4. AI 버튼 선택 (AI가 파라미터를 자동으로 판단하도록 설정)
5. 이름을 "Get Schedule"로 지정

일정 생성 도구:
1. 도구 추가 > Google Calendar > Event > Create 선택
2. 인증 정보 연결
3. 캘린더 선택
4. AI 버튼 선택
5. Additional Fields에서 참석자, 설명 등도 AI로 설정
6. 이름을 "Create Event"로 지정

일정 삭제 도구:
1. 도구 추가 > Google Calendar > Event > Delete 선택
2. 인증 정보 연결
3. 캘린더 선택
4. AI 버튼 선택
5. 이름을 "Delete Event"로 지정

### 2. Google Sheets 툴 설정

연락처 시트 읽기 도구:
1. 도구 추가 > Google Sheets > Sheet > Read 선택
2. 인증 정보 연결
3. 문서와 시트 선택
4. 이름을 "Get Contacts"로 지정

### 3. Gmail 툴 설정

이메일 발송 도구:
1. 도구 추가 > Gmail > Message > Send 선택
2. 인증 정보 연결
3. To, Subject, Message 필드를 AI로 설정
4. Additional Option에서 "Append n8n attribution" 해제
5. 이름을 "Send Email"로 지정

## 시스템 프롬프트 설정

AI Agent 노드의 System Message에 다음과 같은 프롬프트 설정:

```
너는 똑똑하고 유능한 어시스턴트야. 오늘 날짜는 {{ $now.format('yyyy-MM-dd') }}야.

너의 역할은 다음과 같아:

---

🔹 1. 일정 처리 (Google Calendar)
- 사용자가 일정 관련 요청을 하면 아래 순서대로 진행해:
  1. **"Get Contacts"** 도구를 사용해 시트에서 팀원 이메일 주소를 확인해.
  2. **"Get Schedule"** 도구로 오늘 또는 요청된 날짜의 캘린더 일정을 조회해.
  3. 일정이 없다면, **"Create Event"** 도구를 사용해 새 일정을 생성해.
  4. 기존 일정이 있지만 **새 요청이 기존 일정과 다를 경우**, 즉 "일정 변경"이 감지된 경우:
     - **"Delete Event"** 도구를 사용해 기존 일정을 먼저 삭제하고,
     - **"Create Event"** 도구를 사용해 새로운 일정을 생성해.

---

🔹 2. 일정 리마인더 이메일 처리
- 사용자가 새로운 일정을 추가했거나 기존 일정을 수정한 경우,  
  **반드시 다음과 같이 질문해:**

> "추가된 일정을 기반으로 리마인더 이메일을 보내드릴까요?"

- 사용자가 "예", "응", "그래", "좋아" 등 긍정적으로 답하면:
  1. **"Send Email"** 도구를 사용해 리마인더 이메일을 발송해.
  2. 이메일 내용에는 새로 추가되거나 수정된 일정 정보를 요약해서 포함해.

- 사용자가 부정적으로 답하면 아무 작업도 하지 마.

---

🔹 각 도구의 역할 정리:
- **Get Contacts**: 시트에서 팀원 이메일 주소 확인
- **Get Schedule**: 구글 캘린더 일정 조회
- **Create Event**: 새 일정 추가
- **Delete Event**: 기존 일정 삭제
- **Send Email**: 일정 리마인더 이메일 발송

---

🧠 중요한 원칙:
- 사용자의 요청이나 동의 없이 이메일을 임의로 발송하지 마.
- 일정 변경이 있을 경우 반드시 기존 일정을 **Delete Event** 도구로 삭제한 뒤,  
  새로운 일정을 **Create Event** 도구로 다시 추가해.
- 항상 정확하고 필요한 작업만 수행해.
```

### 날짜 포맷 설정 (Formula)

시스템 프롬프트에서 현재 날짜를 표시하기 위한 포맷:
- `{{ $now.format('yyyy-MM-dd') }}` - 현재 날짜를 yyyy-MM-dd 형식으로 표시

## 타임존 설정

워크플로우에서 정확한 현지 시간을 사용하도록 타임존 설정:

1. 워크플로우 설정(Workflow Settings) 메뉴로 이동
2. 타임존을 "Asia/Seoul"로 설정
3. 저장

## 테스트 및 배포

### 내부 테스트

1. "Open chat" 버튼 클릭
2. AI Agent와 대화 시작
3. 워크플로우 실행 경로와 로그 확인

### 공개 링크 생성

1. Chat message received 트리거 설정으로 이동
2. "Make chat publicly available" 옵션 활성화
3. Hosted Chat 선택
4. Authentication 방식 설정 (n8n user auth 권장)
5. Initial Message 설정 (예: "안녕하세요, 자비스입니다. 어떤 도움이 필요하신가요?")
6. 생성된 URL을 사용하여 외부에서 접근 가능

### 테스트 예시 문구

- "내일 내 일정을 확인해서 브리핑해줘."
- "오후3시에서 4시까지 홍길동과 AI Agent 구축 미팅이 있어. 참석자로 홍길동도 넣어서 일정을 추가해줘."
- "리마인더 이메일도 보내줘"

---

이 가이드를 통해 n8n에서 코딩 없이 AI Agent를 설정하고 사용하는 방법을 배울 수 있습니다. AI Agent는 복잡한 워크플로우 없이도 유연하게 사용자 요청에 대응할 수 있는 강력한 도구입니다.