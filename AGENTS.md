# AGENTS.md — AI 에이전트·코딩 어시스턴트를 위한 안내

**`kea-reg-mcp`는 한국에너지공단(KEA) 현행 규정을 조문 단위로 검색하는 MCP 서버입니다.** ([koica-reg-mcp](https://github.com/amnotyoung/koica-reg-mcp) 포크)

규정 데이터는 저장소에 포함되지 않습니다. 처음 한 번 `alio_sync.py`로 ALIO에서 받아야 합니다.

## 이 저장소를 "사용"하려는 경우

```bash
git clone https://github.com/hsangjin0315-sudo/kea-reg-mcp.git
cd kea-reg-mcp
pip install -r requirements.txt
python3 alio_sync.py      # ALIO에서 현행 규정을 받아 추출·색인 (Node.js 필요, 수 분)
```

**Claude Code**
```bash
claude mcp add kea-reg -- /절대/경로/python3 /절대/경로/kea-reg-mcp/kea_mcp_server.py
```

**Claude Desktop** — `claude_desktop_config.json`에 등록 (README 참고).

제공 도구 11종: `search_regulation`, `get_article`, `verify_citation`,
`find_references`, `compliance_radar`, `list_attachments`, `get_attachment`,
`list_sources`, `find_questions`, `sync_from_alio`, `update`.

## 이 저장소를 "개발·수정·기여"하려는 경우

- `kea_mcp_server.py` — 로컬 stdio 서버 (도구 11개)
- `server_http.py` — 원격 streamable-http 서버 (선택, `MCP_SHARED_SECRET` 설정 시 `x-mcp-key` 헤더 인증)
- `alio_sync.py` — ALIO에서 현행 규정 수집(HWP 다운로드 → kordoc 변환 → 정규화 → 색인). Node.js 필요.
- 검색 엔진(`kea_search.py`)은 순수 표준 라이브러리(런타임 의존성은 `mcp` 하나).
- `data/` 아래는 전부 gitignore — 규정 데이터를 커밋하지 마십시오(NOTICE 참고).
