# 구조

## 1. 무엇을 하나

고객(지역 공공사업 조직)의 정보형 뉴스레터 한 호를 만드는 단계별 스크립트 묶음. 사람(또는 Claude)이 단계 사이에서 후보를 고르고 원고를 쓰며, 스크립트는 **결정적인 일**(일정 · 수집 · 거르기 · 검사 · 렌더)을 맡는다.
각 단계는 `python -m lf.<이름> <slug> <발송일> …` 로 돌고, 결과를 호 폴더의 파일로 남긴다. 단계끼리는 파일로만 주고받는다.

## 2. 호 하나의 흐름

```
new_issue      issue.yaml (영업일 일정 D-5 … D, 마감 기준일 = 발송일 + 1)
   │
   ├─ naver_news   candidates_news.json   (네이버 검색 API — 2026-10-12 이후 레터팩토리에서는 안 씀)
   ├─ boards       candidates_boards.json, boards/NN.html|json   (sources.yaml 의 게시판)
   ├─ blog         (블로그 본문 — 안 씀, 법적 위험)
   └─ prev_issue   prev_issue_urls.txt    (지난 호에 실린 링크 → 제외 목록)
   │
digest         digest_raw.md          (후보를 축별로 묶어 한 번에 읽게)
quality        quality.json · quality.md · digest_checked.md   (길이 · 민감어 · 제목 · 중복)
   │
(사람 / Claude)  selection.md → draft.md   (선별 · 원고 — 스크립트 아님)
   │
checks         checks.json            (링크 · 이미지 열림, 마감 지남, 금지 표현, 중복, 미완성 표시, data: 이미지, 제목) — 고칠 것이 있으면 exit 1
render         letter.html            (draft.md → 인라인 스타일 · 표 레이아웃 이메일 HTML, 템플릿 선택)
preview        데스크톱 · 휴대폰 스크린샷
stibee         stibee.html (스티비 붙여 넣기용), 구독자 동기화, 테스트 메일 링크 검사
sync           데이터 폴더(구글 드라이브) 복사 기록
status         지금 어느 단계인지 · 다음 명령
env_check      이 세션에서 파이프라인을 돌릴 수 있는지 (Cowork 세션 시작 때)
```

## 3. 모듈

| 모듈 | 역할 | TS 플랫폼으로 옮김 |
|---|---|---|
| `common` | 데이터 폴더 경로(`LF_ROOT`) · `.env` · YAML · 텍스트 정리 · 한국 영업일 | ✅ (경로 · `.env` 는 CLI 쪽으로) |
| `new_issue` | 호 폴더와 영업일 일정. 기존 `issue.yaml` 은 보존하고 빈 칸만 채움 | ✅ |
| `boards` | 게시판 목록에서 날짜 있는 글 링크 추출. `goDetail` 주소 조립, 기간 · 재게재 후보, 게시판별 필터, `legacy_tls`(옛 TLS 사이트), 인증서 무시 | ✅ |
| `prev_issue` | 지난 호(붙여 넣은 텍스트)에서 링크 제외 목록 | ✅ |
| `digest` | 후보 다이제스트 | ✅ |
| `quality` | 품질 거르기 (민감어 · 제목 · 유사도) | ✅ |
| `checks` | 원고 검사 | ✅ |
| `render` | 이메일 HTML. Jinja2 템플릿 + 템플릿별 인라인 스타일 YAML, `:::card` 2단 카드 | ✅ |
| `naver_news` · `blog` | 네이버 수집 | ❌ 네이버 제외 결정 |
| `stibee` · `sync` · `mcp_server` · `env_check` · `status` · `preview` | 발송 도구 · 드라이브 · Claude 데스크톱 연결 · 운영 보조 | ❌ 플랫폼에서 새로 만든다 |

템플릿: `lf/templates/<이름>.html`(머리 · 꼬리 틀) + `<이름>.yaml`(본문 인라인 스타일). public · modern · warm · colorful 4종 + 자체 뉴스레터 전용 1종. 공휴일: `lf/config/holidays_kr.yaml`(해가 바뀌면 추가해야 한다).

## 4. 데이터 폴더

코드와 분리돼 있다. 구조와 `draft.md` 형식은 루트 [`README.md`](../README.md) 「데이터 폴더 구조」 · 「draft.md 형식」.
고객별 `customers/<slug>/profile.yaml`(고객 정보 · 어조 · 코너 · 템플릿 · 브랜드) · `sources.yaml`(검색어 · 게시판 · 품질 규칙), 호마다 `customers/<slug>/issues/<발송일>/`(또는 `lf_settings.yaml` 의 `issue_dir` 규칙).

## 5. MCP 서버

`lf/mcp_server.py` (stdio). Claude 데스크톱 · Cowork 가 데이터 폴더에서 `lf_run`(단계 실행) · `lf_status` · `read_text` · `write_text` · `list_dir` 를 쓰게 한다. 긴 단계는 백그라운드로 띄우고 바로 `run_id` 를 돌려준다(Cowork 브리지가 도구 호출을 약 60초만 기다리기 때문). `.env` 읽기 · 데이터 폴더 밖 쓰기는 거부. 자세한 표는 루트 README.
