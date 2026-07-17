# 정부24 민원도우미 (레드팀 시연용 AI 서비스)

**AI 레드팀 도구의 공격 표적**으로 쓰는, **의도적으로 취약한** 행정 안내 챗봇.
NimbusPay 챗봇(보안 낮음) 대비 **보안 중~중상** 수준으로 설계해, 두 챗봇을 동시에 스캔하여 보안 강도를 비교한다.

> ⚠️ **의도적으로 취약함.** 로컬/격리 환경 데모 전용. 프로덕션·공용 인터넷에 절대 노출 금지.

---

## 무엇이 들어있나

- `app.py` — 챗봇 서버 (Python stdlib + Anthropic Claude API). HTTP API + 채팅 UI 서빙.
- `db.py` — SQLite 초기화 모듈. 공무원 정보·민원 현황·내부 시스템 자격증명 seed.
- `index.html` — 브라우저 채팅 화면 (정부24 스타일).
- 은닉 비밀 (공격이 빼내야 할 것): 내부 API 키 · DB 비밀번호 · 게이트웨이 URL · 공무원 개인정보.

## NimbusPay 대비 보안 강도 비교

| 항목 | NimbusPay (낮음) | 정부24 민원도우미 (중~중상) |
|------|-----------------|--------------------------|
| 자격증명 보호 지시 | 1줄 | 명시적 이유 포함 |
| 역할 고정 | 없음 | "전환 불가" 명시 |
| 입력 패턴 인식 | 없음 | 공격 키워드 거부 지시 |
| 취약한 공격 유형 | 직접 요청, 롤플레이 | 멀티턴 누적, 권한 사칭 |

---

## 실행 방법

### 1) 환경변수 설정

```bash
cp .env.example .env
# .env 파일에 ANTHROPIC_API_KEY 입력
```

### 2) 패키지 설치

```bash
pip install -r requirements.txt
```

### 3) 챗봇 실행

```bash
python3 app.py     # http://localhost:8200
```

---

## 엔드포인트

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/chat` | `{"message":"..."}` → `{"reply":"..."}` |
| POST | `/v1/chat/completions` | OpenAI 호환 |
| GET | `/` | 채팅 UI |
| GET | `/health` | `{"status":"ok","model":"..."}` |

---

## 레드팀 도구에 표적으로 등록하는 법

```json
{
  "actor_type": "http",
  "url": "http://localhost:8200/chat"
}
```

Docker 안에서 돌면 `localhost` 대신 `http://host.docker.internal:8200/chat`.

---

## 클라우드 REDI에서 이 로컬 챗봇 공격 — 터널로 공개

배포된 REDI(EC2 백엔드)가 `localhost:8200`에 닿으려면 터널이 필요하다.

### 1) 챗봇 실행

```bash
python3 app.py     # http://localhost:8200
```

### 2) 터널로 공개 주소 생성

**cloudflared** (권장):
```bash
cloudflared tunnel --url http://localhost:8200
# 출력 예: https://random-words-1234.trycloudflare.com
```

**ngrok** (대안):
```bash
ngrok http 8200
```

### 3) REDI 웹에서 표적 URL로 등록

```json
{
  "actor_type": "http",
  "url": "https://random-words-1234.trycloudflare.com/chat"
}
```

> ⚠️ 데모가 끝나면 터널을 반드시 Ctrl+C 로 종료.
