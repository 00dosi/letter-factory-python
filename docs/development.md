# 개발 환경 · 테스트

## 1. 필요한 도구

| 도구 | 버전 | 비고 |
|---|---|---|
| Python | **3.12 이상** | `pyproject.toml` 은 `>=3.10` 이라고 적혀 있지만 `lf/stibee.py` 52행이 f-string 안에 역슬래시를 써서 **3.10 · 3.11 에서는 문법 오류**가 난다 → [status.md](status.md) |
| [uv](https://docs.astral.sh/uv/) | 0.12 대에서 확인 | 가상환경 · 의존성 · 테스트 실행 |
| 의존성 | `requests` · `PyYAML` · `Jinja2` · `Pillow` | `pyproject.toml`. 버전 고정 없음 |
| Windows Chrome | — | `lf.preview`(스크린샷)만. WSL 에서 Windows 쪽 chrome.exe 를 부른다 |

## 2. 설치

```bash
git clone https://github.com/00dosi/letter-factory-python
cd letter-factory-python
```

따로 `pip install` 하지 않고 `uv run` 으로 돌린다. 사용자 PC 에 MCP 서버로 까는 방법은 루트 [`README.md`](../README.md) 「설치」.

## 3. 환경 변수 · 비밀값 (이름만)

코드 폴더가 아니라 **데이터 폴더**의 `.env` 에 둔다(`lf/common.py` `load_env`). 셸 환경변수 `NAVER_*` · `STIBEE_*` 가 있으면 그것이 이긴다.

| 이름 | 쓰는 모듈 | 어디서 받나 |
|---|---|---|
| `NAVER_CLIENT_ID` · `NAVER_CLIENT_SECRET` | `naver_news` | 네이버 개발자센터 애플리케이션(검색 API). 레터팩토리는 2026-10-12 부터 네이버 수집을 쓰지 않기로 했다 |
| `STIBEE_API_KEY` · `STIBEE_LIST_ID` (선택) | `stibee` | 스티비 계정 설정 |
| `LF_ROOT` | 전체 | 데이터 폴더 경로. 없으면 현재 폴더. MCP 서버는 `--root` 로 넣는다 |

## 4. 테스트 · 실행

2026-10-14, `main`(7c51d68) 을 새로 받은 작업 폴더에서 직접 돌린 결과.

| 명령 | 결과 |
|---|---|
| `uv run --python 3.12 --with pytest pytest tests -q` | ✅ **74 passed** |
| `uv run --with pytest pytest tests -q` (uv 가 3.11 을 고른 경우) | ❌ 수집 단계 오류 3건(`test_render_card` · `test_render_dosirak` · `test_stibee_testmail`) — 위 `stibee.py` 문법 문제. Python 3.12+ 로 돌린다 |
| `LF_ROOT=<가상 데이터 폴더> PYTHONPATH=<체크아웃> uv run --python 3.12 --no-project --with requests,PyYAML,Jinja2,Pillow python -m lf.render <slug> <발송일>` | ✅ `letter.html` 생성 (가상 고객 데이터로 확인) |
| 같은 방식 `python -m lf.status <slug>` | ✅ 진행 단계 · 다음 명령 출력 |
| `uv run lf/mcp_server.py --root <데이터 폴더>` | 실행 안 해 봄 (stdio MCP 서버라 입력을 기다리며 멈춘다) |
| `naver_news` · `boards` · `blog` · `stibee` · `preview` | 실행 안 해 봄 (바깥 네트워크 · API 키 · Windows Chrome 필요) |

주의:

- `uv run` 이 체크아웃에 `uv.lock` · `.venv` 를 만든다. `.venv` 는 `.gitignore` 에 있지만 **`uv.lock` 은 커밋하지 않는다**(지운다).
- 실제 호 폴더를 덮지 않게, 시험은 데이터 폴더를 임시 폴더에 **복사**해서 `LF_ROOT` 로 돌린다.
- 테스트 픽스처 `tests/fixtures/regression_*.html` 은 템플릿 4종(warm · modern · public · colorful)의 출력을 바이트 단위로 고정한다. 이 파일이 바뀌면 "다른 템플릿은 그대로" 규칙을 깬 것이다. TS 플랫폼의 골든 테스트도 이 출력을 정답으로 쓴다.
- lint · 포맷 도구는 설정돼 있지 않다.

## 5. 변경을 내보내는 법

- 지금까지는 `main` 에 직접 push 했고 PR · CI 는 없었다. 2026-10-14 인수인계 문서부터 PR 로 올린다.
- 사용자 PC 의 MCP 서버는 `main` zip 을 받아 쓰므로, **push 하면 바로 사용자 쪽 다음 `--refresh` 에 반영된다.** 사용자 PC 갱신 명령은 루트 README 「설치」 4번.
- 동작을 바꾸면 TS 이식 쪽 골든 기대값도 다시 만들어야 한다(TS 저장소의 `golden:regen` 이 이 체크아웃을 읽는다).
