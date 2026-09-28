# piia-engram 분석 정리 (한국어)

> 작성일: 2026-09-28 · 분석 대상 버전: v4.20.0
>
> - 원본 레포: https://github.com/Patdolitse/piia-engram
> - 포크 레포: https://github.com/bmshin94/piia-engram
> - PyPI: https://pypi.org/project/piia-engram/

이 문서는 레포 전수조사 후 나눈 대화(1~4번 질문)를 정리한 것이다.

---

## 1. 전수조사 분석: 무엇을 하는 프로젝트인가

### 한 줄 요약

AI에게 "나"를 한 번만 설명하면 Claude Code, Cursor, Codex 등 여러 AI 코딩 도구가
똑같이 기억하게 해 주는 **로컬 우선(local-first) AI 개인 기억/정체성 저장소**다.

- 원작자: Patdolitse (`NOTICE` 기준 Copyright 2026)
- 언어: Python 3.10+ · 라이선스: AGPL-3.0-or-later · 패키지명: `piia-engram`
- 규모: 파일 약 640개, 파이썬 소스 약 5.9만 줄, 테스트 약 4,900개
- 포크본은 원본 v4.20.0 그대로이며, 추가된 것은 `CLAUDE.md`(페르소나 가이드)뿐이다.

### 해결하는 문제

- 새 채팅을 열거나 툴을 바꾸면 AI의 기억이 초기화된다.
- 각 툴의 기억(Claude Memory, Cursor Rules 등)은 그 툴 안에 갇혀 있다.
- engram은 `~/.engram/`에 JSON/Markdown 파일로 사용자 정보를 저장하고,
  모든 툴이 **MCP**(Model Context Protocol)로 같은 내용을 읽게 한다.
  클라우드 계정 없음, 기본 네트워크 호출 0.

### 저장 구조 (`~/.engram/`)

| 폴더 | 내용 |
|---|---|
| `identity/` | 프로필, 선호, 품질 기준, 비공개 필드 설정 |
| `knowledge/` | 교훈(lessons), 결정(decisions), 도메인 지식 |
| `playbooks/` | 다단계 작업 절차서 |
| `projects/` | 프로젝트 스냅샷 |
| `contexts/` | 툴별 세션 기록(이어하기용) |
| `tools/` | 로컬 설치 프로그램/CLI 위치 지도 |

### 레포 폴더별 역할

| 폴더 | 역할 |
|---|---|
| `src/piia_engram/` | 본체. `mcp_server.py`가 MCP 도구 59개 노출, `core.py`(Engram 클래스)+믹스인이 로직, `storage.py`가 원자적 파일 I/O |
| `src/piia_engram/hooks/` | Claude Code/Cursor 훅: 세션 시작 시 이어하기 브리프 자동 주입, 종료·압축 시 자동 저장 |
| `src/piia_engram/watcher/` | 훅이 없는 툴용 폴링 감시자(대화 기록 파일 → 세션 체크포인트) |
| `src/piia_engram/dock_ui/` | 로컬 웹 관리 대시보드(`engram serve --ui`) |
| `src/piia_engram/embedded/` | 다른 앱에 내장할 때의 계약 |
| `skills/engram/` | 언제 어떤 도구를 호출할지 안내하는 스킬 |
| `.claude-plugin/`, `.cursor-plugin/`, `.mcp.json` | Claude Code / Cursor 플러그인 매니페스트 |
| `tests/` | 테스트 스위트 |
| `docs/` | 사용자 가이드, 아키텍처, 보안/신뢰, 비교, 벤치마크, 설계 스펙 |
| `demos/` | 툴 간 연속성 증명 데모·벤치마크 |
| `scripts/` | 릴리스 검증, 문서 수치 동기화 검사 등 유지보수 스크립트 |
| `worker/` | 선택형 익명 통계용 Cloudflare Worker(기본 꺼짐) |
| `examples/` | `~/.engram` 예시 JSON |
| `release-evidence/` | 버전별 릴리스 검증 기록 |

### 핵심 기능

1. 세션 시작 자동 로드: `get_user_context`, `get_resume_brief`
2. 지식 저장: `add_lesson`, `add_decision`, `add_playbook`, `wrap_up_session`
3. 검색: 키워드 기본, 선택 시 하이브리드(FTS5 + 벡터, 다국어 교차 검색)
4. 거버넌스: 고위험 항목(자격증명, 셸 명령, 권한 규칙)은 staging에서 승인 대기, 민감정보 자동 마스킹
5. 플레이북 자동 추출: 다단계 작업 종료 시 초안 생성 → 사용자 확인 후 확정
6. Memory Lens: `engram preview --html`로 AI가 받을 내용 미리보기
7. MCP 미지원 툴(ChatGPT, Gemini 등)용 Markdown 신분증 카드 내보내기
8. 로컬 감사 로그, 선택형 AES-256-GCM 필드 암호화

### 언제 쓰나

- 여러 AI 코딩 툴을 번갈아 쓸 때
- 새 채팅마다 같은 설명을 반복할 때
- 과거 결정의 이유를 자주 찾을 때
- 반복 작업 절차를 AI가 기억해 주길 원할 때

### 나에게 주는 도움

- 한국어·코드 스타일·기술 스택(React/PHP) 선호를 모든 AI 툴에 일괄 적용
- 프로젝트 간 교훈·결정 누적 → 새 프로젝트를 과거 경험 위에서 시작
- 데이터가 로컬 파일이라 직접 확인·수정 가능, 회사 작업에도 부담이 적음
- MCP 서버 설계, 훅, 테스트 문화를 배울 수 있는 좋은 참고 코드

---

## 2. 더 쉬운 설명

AI 툴을 **퇴근하면 기억이 지워지는 알바생**, engram을 **카운터의 업무 수첩**으로 비유할 수 있다.

- 1페이지 "사장님은 한국어 선호" = identity
- 2페이지 "냉장고 문 안 닫아서 재료 버림, 꼭 닫을 것" = lesson
- 3페이지 "원두는 A사로 결정, B사는 비싸서 탈락" = decision
- 4페이지 "마감 순서: 정산 → 청소 → 문단속" = playbook

어떤 알바생(툴)이 와도 출근하자마자 수첩을 읽는다. 수첩은 가게 안(내 PC)에만 있고,
위험한 내용은 "사장님 확인 대기" 칸에 들어가며, 언제든 직접 고치고 지울 수 있다.

하루 흐름:

1. Claude Code 실행 → 훅이 "어제 로그인 API 리팩토링 중이었음"을 자동 주입
2. 작업 중 "원인은 타임존이었어, 기억해 줘" → 교훈 저장
3. 종료 시 세션 요약 자동 저장, 반복 작업이면 플레이북 초안 생성
4. 다음 날 Cursor를 켜도 같은 기록과 교훈을 알고 있음

구성품 비유: MCP 서버 = 창구, 훅 = 자동 스위치, 워처 = CCTV, Dock UI = 관리자 화면,
스킬 = 안내문, CLI = 건강검진 도구.

---

## 3. 질문별 답변

### 3-1. 설치 및 사용법

```bash
pip install piia-engram
engram setup            # AI 툴 자동 감지 → 설정 백업 후 MCP 연결 → 툴 재시작
```

- Claude Code 수동: `claude mcp add piia-engram -- piia-engram-mcp`
- 무설치(uv): `"command": "uvx", "args": ["--from", "piia-engram", "piia-engram-mcp"]`
- 포크 소스 설치: `git clone https://github.com/bmshin94/piia-engram && cd piia-engram && pip install -e .`
- Cursor: `~/.cursor/mcp.json`에 `{"mcpServers": {"piia-engram": {"command": "piia-engram-mcp"}}}`

사용은 AI에게 자연어로 말하면 된다: "지난번 이어서 하자"(`get_resume_brief`),
"이거 기억해 줘"(`add_lesson`), "DB 왜 이걸로 정했지?"(`search_knowledge`).

| CLI | 기능 |
|---|---|
| `engram doctor` / `--fix` | 진단 / 자동 수리 |
| `engram status --html` | 상태 페이지 |
| `engram preview --html` | AI가 받을 내용 미리보기 |
| `engram review` | 승인 대기 지식 관리 |
| `engram serve --ui` | 웹 대시보드(`piia-engram[ui]` 필요) |
| `engram export-agents-md` | 검증 지식을 AGENTS.md/CLAUDE.md 블록으로 내보내기 |

주요 환경변수: `ENGRAM_TOOLS=all`, `ENGRAM_SEARCH=hybrid`, `ENGRAM_APPROVAL=strict`, `ENGRAM_GOVERNANCE=1`.

### 3-2. 플러그인인가, 스킬인가, MCP인가

본체는 **MCP 서버**이고, 플러그인과 스킬은 이를 감싼 배포 형태다.

| 형태 | 위치 | 역할 |
|---|---|---|
| MCP 서버(본체) | `src/piia_engram/mcp_server.py` | 도구 59개(기본 19 + 고급 40) |
| Claude Code 플러그인 | `.claude-plugin/plugin.json`, `.mcp.json` | 설치 시 MCP 서버 등록 |
| Cursor 플러그인 | `.cursor-plugin/plugin.json` | 스킬 + MCP 묶음 |
| 스킬 | `skills/engram/SKILL.md` | 도구 호출 시점 안내(라우팅) |
| 훅 | `src/piia_engram/hooks/` | 세션 시작/종료 자동 실행 |
| CLI | `engram` | 설치·진단·관리 |

### 3-3. API 토큰이 필요한가

LLM API 키는 **필요 없다**. 소스에 LLM API 호출 코드가 없으며, 추론은 사용 중인 AI 툴이 담당한다.
선택 항목만 있다.

- `ENGRAM_AUTH_TOKEN`: 원격(SSE) 모드용, 사용자가 직접 생성하는 접속 토큰
- `ENGRAM_SECRET`: 필드 암호화용 패스프레이즈
- 하이브리드 검색(`[vector]`)은 임베딩 모델을 최초 1회 다운로드(약 230~280MB) 후 로컬 실행

### 3-4. 깃허브에서 인기 있는 이유(분석)

정확한 스타 수는 확인하지 않았고, 코드·문서에서 보이는 요인을 정리했다.

1. "AI가 매번 잊는다", "툴마다 기억이 따로"라는 보편적 문제 해결
2. MCP 표준화 흐름에 맞춘 "모든 툴 공용 기억" 포지션
3. 로컬 우선·무계정·파일 직접 열람으로 높은 신뢰
4. `pip install` + `engram setup` 두 줄 설치
5. 공식 MCP Registry, awesome-mcp-servers, Glama, Cursor Directory, Smithery 등 등재
6. 약 4,900개 테스트, 릴리스 증거 기록, 문서 수치 CI 검사 등 높은 엔지니어링 품질
7. 중국어 문서·샤오홍슈 이미지·중국 AI IDE 지원 등 커뮤니티 마케팅
8. 경쟁 제품과의 솔직한 비교 문서

### 3-5. 로컬 에이전트 구축에 도움이 되나

도움이 된다. 에이전트의 **장기기억(사용자 기억) 부품**으로 적합하다.

- 로컬 우선, 네트워크 없이 약 100ms 내 시작
- MCP 표준이라 MCP 지원 에이전트에 바로 연결(Hermes 검증, OpenClaw 파일 연동)
- LLM 독립적: 호스트가 MCP를 지원하면 로컬 LLM 기반 에이전트에도 연결 가능
- `from piia_engram.core import Engram`으로 파이썬 직접 임베드 가능
- staging 승인, 민감정보 마스킹, 감사 로그, 훅/워처 구조는 설계 참고용으로 좋음

한계: "사람에 대한 기억"에 특화되어 있어 대용량 문서 RAG나 작업 로그 저장소로는 부적합하다.

### 3-6. 수익화 아이디어

한국어 교육 콘텐츠, 도입 컨설팅, 팀용 SaaS 자체 개발, 플레이북·템플릿 판매 등(4장 참고).
AGPL-3.0이므로 수정본을 네트워크 서비스로 제공하면 수정 소스 공개 의무가 있다.

### 3-7. React나 PHP로 만들 수 있나

가능하다.

| 부분 | 추천 기술 |
|---|---|
| MCP 서버 | Node.js/TypeScript(공식 MCP SDK) 또는 PHP(MCP PHP SDK) |
| 관리 대시보드 | React |
| 팀/웹 서비스 백엔드 | PHP(Laravel) + MySQL |
| 저장소 | 개인용 JSON/SQLite, 팀용 MySQL |

MVP 순서: TS MCP 서버(도구 3개: `get_user_context`, `add_lesson`, `search_knowledge`) →
JSON 저장 + 키워드 검색 → React 대시보드 → Laravel 팀 공유/로그인.

주의: engram 코드를 그대로 번역하면 파생 저작물로 AGPL이 적용된다. 아이디어만 참고해 직접 설계해야
라이선스를 자유롭게 정할 수 있다.

---

## 4. 수익화 아이디어 상세

### 라이선스 규칙

| 하고 싶은 것 | 가능 여부 | 조건 |
|---|---|---|
| 설치/세팅 대행, 교육 | 가능 | 자유 |
| 수정본을 웹 서비스로 운영 | 가능 | 수정 소스 공개(AGPL 네트워크 조항) |
| 코드를 비공개 상용 제품에 포함 | 불가 | AGPL 위반 |
| 아이디어만 참고해 새로 개발 | 가능 | 자체 라이선스 |
| "piia-engram" 이름으로 판매 | 주의 | 원작자 명칭, 자체 브랜드 권장 |

### 아이디어 1. 한국어 교육 콘텐츠 (난이도 낮음)

- "AI 코딩툴 기억력 10배" 강의·유튜브·전자책
- 한국어 자료가 거의 없어 선점 가능
- 커리큘럼: MCP 개념 → 설치 → Claude Code+Cursor 연동 → 플레이북 → 나만의 MCP 서버(React/TS)
- 수익: 인프런/클래스101, 유튜브 광고, 유료 뉴스레터, 크몽 전자책

### 아이디어 2. 기업 도입 컨설팅/세팅 대행 (난이도 중하)

- 개발팀 AI 툴 환경 표준화: MCP 설정, CLAUDE.md/AGENTS.md 작성, 팀 교훈·플레이북 구축
- 대상: AI 툴 도입 중인 스타트업, 에이전시, SI 업체
- 수익: 건당 세팅비 + 월 유지보수, 사내 워크숍

### 아이디어 3. 팀용 "AI 기억 허브" SaaS (난이도 높음, 잠재력 최대)

- engram은 개인용 → 팀용으로 확장: 팀 공통 교훈/결정/플레이북, 신입 온보딩 자동화,
  관리자 승인 워크플로, 권한·변경 이력
- 스택: React 대시보드 + PHP(Laravel) + TypeScript MCP 서버 + MySQL
- 수익: 인원당 월 구독(무료 3인 → 유료 팀 플랜)
- 차별점: 한국어 우선, 국내 협업툴/Slack 연동, 설치형 옵션
- engram 코드 재사용 없이 직접 설계해야 비공개 상용 가능

### 아이디어 4. 플레이북·템플릿 마켓 (난이도 중하)

- 코드가 아닌 콘텐츠 판매: "Laravel 배포 플레이북", "React 성능 교훈 50선", "PHP 보안 체크리스트"
- engram JSON + CLAUDE.md 형식 동시 제공
- 추후 SaaS 템플릿 스토어로 통합

### 아이디어 5. 무료 웹 도구로 트래픽 확보 (난이도 중하)

- "AI 신분증 카드 생성기": 질문에 답하면 CLAUDE.md / .cursorrules / engram JSON 자동 생성(React)
- 광고 + 이메일 수집 → 강의·SaaS 전환(리드 마그넷)

### 아이디어 6. 오픈소스 기여로 개인 브랜딩 (간접 수익)

- 원본 레포에 한국어 문서 번역 PR → 기여 이력, 포트폴리오, 외주 단가 상승

### 추천 실행 순서

1. 1개월차: 한국어 블로그/유튜브 + 무료 생성기(아이디어 5)
2. 2~3개월차: 컨설팅(아이디어 2)으로 현금 흐름과 고객 니즈 확보
3. 4개월차~: 팀 SaaS(아이디어 3) MVP, 템플릿 판매(아이디어 4) 병행
