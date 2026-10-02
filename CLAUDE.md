# CLAUDE.md — Claude Code / Claude 어시스턴트를 위한 안내

**`kea-reg-mcp`는 한국에너지공단(KEA) 현행 규정 검색 MCP 서버입니다.** (koica-reg-mcp 포크)

## KEA 규정 검색 도구로 쓰려는 경우

```bash
git clone https://github.com/hsangjin0315-sudo/kea-reg-mcp.git && cd kea-reg-mcp
pip install -r requirements.txt && python3 alio_sync.py
claude mcp add kea-reg -- <python3 절대경로> <kea_mcp_server.py 절대경로>
```

규정 데이터는 저장소에 없습니다. `alio_sync.py`(Node.js 필요)가 ALIO에서 받아 색인합니다.

## 개발·기여하려는 경우

자세한 구조는 `AGENTS.md`와 `README.md`를 보세요.
- 로컬 stdio: `kea_mcp_server.py`(도구 11개)
- ALIO 동기화: `alio_sync.py` (Node.js + `npx kordoc` 필요)
- `data/`는 커밋하지 않습니다.
