# kea-reg-mcp

**한국에너지공단(KEA) 현행 규정을 MCP로 검색·조회.** 조문 단위 검색 + 본문 직접 조회 + 인용 검증(제목 환각 탐지) + 상호참조 그래프 + **규정 정비 레이더**를 MCP 호환 클라이언트(Claude Desktop·Claude Code·Codex·Cursor 등)에서 씁니다. 규정은 [ALIO(공공기관 경영정보 공개시스템)](https://www.alio.go.kr/item/itemOrganList.do?apbaId=C0097&reportFormRootNo=21110)에서 사용자가 직접 받아 색인합니다.

[![MCP](https://img.shields.io/badge/MCP-stdio%20%7C%20streamable--http-blue)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> [koica-reg-mcp](https://github.com/amnotyoung/koica-reg-mcp)(KOICA 규정 검색)를 KEA용으로 고친 포크입니다. 한국에너지공단과 무관한 개인 프로젝트입니다.

---

## 무엇을 해결하나

- **참조 시간 단축** — 규정 파일을 하나씩 열어 Ctrl+F 하는 대신, 자연어 한 줄로 조문 단위 검색
- **현행본 기준** — ALIO의 KEA 규정 목록에서 각 규정의 **최신 개정본**만 받아 색인 (개정 이력 zip 묶음도 처리)
- **인용 검증** — 보고서·답변에 들어간 "{규정명} 제N조" 인용이 실제로 존재하는지 교차검증 (LLM 환각 방지)
- **상호참조 추적** — 어떤 조문이 시행세칙·관련 지침의 어디로 연결되는지 양방향 그래프

---

## 규정 데이터에 대하여

이 저장소는 **코드만** 담고 있습니다. 규정 본문은 저작권이 한국에너지공단에 있으므로 포함하지 않고, `alio_sync.py`가 ALIO에 공개된 현행본을 **사용자 컴퓨터로 직접** 내려받아 처리합니다. 자세한 내용은 [NOTICE](NOTICE)를 보세요.

---

## 빠른 시작

### 1. 설치와 첫 동기화

```bash
git clone https://github.com/hsangjin0315-sudo/kea-reg-mcp.git
cd kea-reg-mcp
pip install -r requirements.txt
python3 alio_sync.py          # ALIO 조회 → HWP 다운로드 → 추출 → 색인 (수 분)
```

> 동기화에는 **Node.js**가 필요합니다(HWP 추출에 `npx kordoc` 사용). 검색·조회 자체에는 필요 없습니다.

### 2. CLI로 바로 써보기


```bash
python3 kea_search.py search "인사위원회 구성" --source 인사규정
python3 kea_search.py get "인사규정" "제11조"
python3 kea_search.py verify "인사규정 제11조에 따라 인사위원회를 둔다"
python3 kea_search.py refs "직제규정" "제9조"
```

### 3. Claude Desktop에 연결

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "kea-reg": {
      "command": "/opt/anaconda3/bin/python3",
      "args": ["/절대/경로/kea-reg-mcp/kea_mcp_server.py"]
    }
  }
}
```

> `command`에는 `mcp` 패키지가 설치된 Python의 절대 경로를 적어주세요 (`which python3`로 확인).

Claude Desktop을 **완전 종료(Cmd+Q)** 후 재실행하면 입력창 하단 도구 메뉴에 `kea-reg` 도구가 보입니다.

### 4. Codex에 연결 (선택)

```bash
codex mcp add kea-reg -- /opt/anaconda3/bin/python3 /절대/경로/kea-reg-mcp/kea_mcp_server.py
codex mcp list
```

등록 후 Codex를 재시작하면 `kea-reg` 네임스페이스의 도구가 노출됩니다.

---

## 사용 예시

```
"KEA 인사규정 채용 결격사유 알려줘"
  → search_regulation + get_article 자동 호출, 해당 조문 본문 반환

"이 보고서에 인용된 조문들이 실재하는지 검증해줘"
  → verify_citation, 각 인용을 ok/not_found/unknown_source로 분류

"직제규정 제9조를 인용한 다른 규정 다 찾아줘"
  → find_references, incoming 그래프 반환 (include_mermaid로 시각화도)

"모규정 개정에 뒤처진 시행세칙·지침 있나?"
  → compliance_radar, 정비 검토 대상 목록 반환

"KEA 규정 ALIO에서 최신으로 동기화해줘"
  → sync_from_alio, alio.go.kr에서 현행본을 다시 받아 재인덱싱
```

---

## 도구 11개

| 도구 | 입력 | 반환 |
|---|---|---|
| `search_regulation` | `query, category?, source?, limit?, fuzzy?, include_attachments?` | 조문 단위 검색 (`fuzzy`=음절 bi-gram, `include_attachments`=별표·별지 포함) |
| `get_article` | `source, article` | 정확한 조문 본문 전체 |
| `verify_citation` | `text` | 인용 실재성 검증 + **제목 환각 탐지**(존재하는 조문에 엉뚱한 제목 → `content_mismatch`) |
| `find_references` | `source, article, include_mermaid?` | 정방향·역방향 인용 관계 (`include_mermaid`=flowchart 코드 동봉) |
| `compliance_radar` | `source?` | **규정 정비 레이더** — 시행세칙·지침이 모규정 개정에 뒤처졌는지 대조 |
| `list_attachments` | `source?, category?, kind?, include_deleted?` | 별표·별지 목록 (행정처분 기준표, 서식 등) |
| `get_attachment` | `source, label` | 별표·별지 본문 전체 (예: `"별표 1"`, `"[별지 제3호 서식]"`) |
| `find_questions` | `query?, question_id?` | (선택) 시험문제 데이터 + 근거 조문 매핑 — 데이터는 저장소에 없으며 `data/questions.json`을 직접 둔 경우에만 동작 |
| `sync_from_alio` | `timeout_sec?` | **ALIO에서 현행 규정 재수집 → 다운로드 → 추출 → 재빌드** |
| `update` | — | `git pull + build`로 코드 갱신, 변경 요약 + OS별 재시작 안내 반환 |
| `list_sources` | `category?` | 인덱싱된 규정 목록과 조문 수 |

> `category`는 규정 유형(규정/시행세칙/지침/기준/정관) 필터입니다. 대부분은 `source`(규정명)나 본문 검색이 더 정확합니다.

---

## 규정 정비 레이더 (`compliance_radar`)

**"모규정이 개정됐는데 시행세칙/지침은 아직 옛날 그대로 아닌가?"** — 규정 담당자가 반복하는 정비 점검을 한 번의 호출로. 각 하위 규정(시행세칙·지침)의 모(母)규정을 이름 규칙·제1조 인용으로 찾아 개정일을 대조하고, 모규정이 더 최근이면 정비 검토 대상으로 플래그합니다.

한국 조례 정비 관행(상위법 개정 추적)을 KEA 규정 체계(규정 ↔ 시행세칙 ↔ 지침)에 옮긴 기능입니다.

---

## 최신 상태 유지

규정은 수시로 개정됩니다. 주기적으로 다시 동기화하세요.

```bash
python3 alio_sync.py          # 바뀐 규정만 새로 받음 (파일 번호 단위 캐시)
python3 alio_sync.py --fresh  # 캐시를 무시하고 처음부터
```

- 채팅에서 "KEA 규정 ALIO에서 최신으로 동기화해줘" → `sync_from_alio` 도구가 같은 일을 합니다.
- 정기 실행 예(cron, 매월 1일): `0 8 1 * * cd /경로/kea-reg-mcp && rm -f data/_cache/regulations.json data/_cache/resolved.json && python3 alio_sync.py`
  (`alio_sync.py`는 목록 조회 결과를 캐시하므로, 정기 실행 때는 목록 캐시를 지워야 새 개정을 잡습니다.)
- 코드 업데이트는 `git pull` 또는 `update` 도구로 받습니다.

**동기화 파이프라인** (`alio_sync.py`):
`ALIO 목록 조회 → 각 규정의 현행본 해석 → HWP 다운로드(개정 이력 zip이면 최신본 선택) → kordoc로 Markdown 추출 → 정규화 → data/extracted/*.md + sources.json → 색인 재빌드`

---

## 파일 구조

```
alio_sync.py           # ALIO 동기화 파이프라인
kea_search.py           # 색인·검색 엔진 + CLI
kea_mcp_server.py       # MCP 서버 (stdio, 도구 11개)
server_http.py         # 원격 MCP 서버 (streamable-http, 선택)
usage_stats.py         # 도구별 호출 횟수 집계 (선택, 개인정보 미수집)
data/                  # 동기화 결과 — 전부 gitignore
├── extracted/         #   규정별 .md 추출본
├── sources.json       #   규정 매니페스트 (이름·유형·개정일·fileNo)
├── index.json         #   검색 색인
└── _cache/            #   HWP·원시 md 캐시
```

---

## 오류 신고·기여

- 동기화 후에도 빠진 규정, 조문 누락·표 깨짐 → [이슈](../../issues/new?template=regulation-update.md)
- 검색 정확도·버그 → [이슈](../../issues/new?template=bug-report.md)
- 공통 개선(검색 엔진·동기화)은 원본 [koica-reg-mcp](https://github.com/amnotyoung/koica-reg-mcp)에도 함께 제안해 주시면 다른 기관 포크까지 혜택을 봅니다.

---

## 함께 쓰면 좋은 도구

규정에 자주 등장하는 외부 법령(공공기관운영법 등)은 [`korean-law-mcp`](https://github.com/chrisryugj/korean-law-mcp)를 함께 쓰면 한 번의 질문에 KEA 규정 + 한국 법령을 묶어 답할 수 있습니다.

---

## 기술 메모

- **검색 엔진**: 순수 Python 표준 라이브러리 (런타임 의존성: `mcp` 하나)
- **동기화**: ALIO REST(JSON) 조회 + [`kordoc`](https://www.npmjs.com/package/kordoc)(HWP→Markdown, Node)
- **검색 알고리즘**: 한국어 불용어 제거 + 토큰 IDF 가중 부분일치 + 음절 bi-gram fuzzy
- **별표·별지 분리 색인**: 본문 검색에서 서식·기준표 노이즈 제거

---

## 라이선스

- **코드**: MIT License (→ [LICENSE](LICENSE)). 원저작자 amnotyoung, KEA 포크 수정 hsangjin0315-sudo.
- **규정 데이터**: 저장소에 포함되지 않습니다. 동기화로 받은 데이터의 저작권은 한국에너지공단에 있습니다.

> 이 도구는 참조·업무 보조용이며 정확성을 보증하지 않습니다. **법적 효력이 있는 공식·최신 규정본은 반드시 [ALIO](https://www.alio.go.kr) 또는 KEA 공식 채널에서 확인**하세요.
