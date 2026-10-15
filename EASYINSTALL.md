# Easy Install — Muse 오케스트레이션 원스톱 세팅

이 파일 하나로 Muse 오케스트레이션의 설치·로그인·VM 유지를 모두 설정한다.
각 섹션의 코드 블록 우측 상단 복사 버튼 한 번으로 프롬프트 전체가 복사된다.

---

# 1. 도구 설치 프롬프트

아래 코드 블록의 복사 버튼을 눌러 전체를 복사한 뒤 Muse에게 붙여넣으면 된다.
구독하지 않는 도구가 있으면 SKIP 목록에 적어서 빼고 설치할 수 있다.

```
Muse 오케스트레이션 도구를 설치해줘. 아래 순서대로 진행하고, 각 단계마다 결과를 보고해줘.

SKIP: []  <- 구독하지 않는 도구가 있으면 여기에 적어줘. 예: [claude, agy]
(SKIP에 적힌 도구는 설치하지 않고 건너뛴다. ego-light는 구독과 무관하므로 항상 설치한다.)

[1] 사전 확인
- node --version으로 Node.js 18 이상인지 확인
- which claude codex agy로 이미 설치된 게 있는지 확인
- 이미 설치된 건 건너뛰고, 없는 것만 설치해
- SKIP에 있는 도구는 확인도 하지 말고 건너뛰어

[2] Claude Code 설치 (SKIP에 claude가 있으면 건너뜀)
- npm i -g @anthropic-ai/claude-code
- 설치 후 claude --version으로 확인해줘.

[3] Codex CLI 설치 (SKIP에 codex가 있으면 건너뜀)
- npm i -g @openai/codex
- 단, /opt/hatch-image/bin/codex에 이미 있으면 npm 설치를 건너뛰고 그걸 사용해.
- 설치 후 codex --version으로 확인해줘.

[4] Antigravity CLI 설치 (SKIP에 agy가 있으면 건너뜀, Gemini CLI의 공식 후속)
- curl -fsSL https://antigravity.google/cli/install.sh | bash
- ~/.local/bin/agy에 설치됨. PATH에 ~/.local/bin이 없으면 ~/.bashrc에 추가해줘.
- 설치 후 agy --version으로 확인해줘.

[5] pproxy 설치 (브라우저용, 항상 설치)
- pip install --break-system-packages pproxy
- 설치 후 127.0.0.1:3129에서 실행 중인지 확인하고, 안 떠 있으면 시작해줘:
  nohup /usr/local/bin/pproxy -l http://127.0.0.1:3129 -r http://hatch-egress-proxy:3128 >/tmp/pproxy.log 2>&1 &

[6] ego light 설치 (브라우저용, 항상 설치)
- ~/workspace/ego-lite-linux에 이미 있으면 건너뛰고, 없으면 사용자에게 알려줘 (별도 설치 필요)
- ~/.local/bin/ego-browser 실행 파일이 있는지 확인
- ~/.local/bin/chrome-no-sandbox 크롬 래퍼가 있는지 확인 (없으면 사용자에게 알려줘)
- ~/.bashrc에 아래가 있는지 확인하고 없으면 추가해줘:
  export PATH="$HOME/.local/bin:$PATH"
  export EGO_HEADLESS=1
  export EGO_CHROME_PATH=$HOME/.local/bin/chrome-no-sandbox
- ego-browser --doctor로 데몬/크롬 상태를 확인해줘

[7] 최종 보고
아래 형식으로 결과만 간단히 보고해줘:
- claude: 버전 / 설치됨 / 이미 있었음 / SKIP됨
- codex: 버전 / 설치됨 / 이미 있었음 / SKIP됨
- agy: 버전 / 설치됨 / 이미 있었음 / SKIP됨
- pproxy: 실행 중 또는 시작함
- ego-light: 정상 / 문제 있음 (상세)

로그인은 내가 직접 할 테니 설치만 하고 멈춰줘.
```

---

# 2. 로그인 안내 (설치 후 사용자가 직접, SKIP한 도구는 제외)

## Claude Code

    claude auth login

1번(Claude 구독 계정)을 선택하면 인증 URL이 표시된다. 브라우저가 있는 기기에서 URL을 열고 Authorize를 클릭한 뒤, 받은 승인 코드를 터미널에 붙여넣는다.

확인: `claude auth status --text` → `Login method`가 표시되면 정상.

주의: VM 공유 IP에서 HTTP 429가 나면 개인 네트워크 기기에서 로그인 후 `~/.claude/.credentials.json`을 파일로 옮긴다. 승인 코드는 1회용이다.

## Codex CLI

    codex login

ChatGPT 계정으로 OAuth 로그인. device-auth 방식도 지원한다.

확인: `codex login status` → `Logged in using ChatGPT`라고 표시되면 정상.

주의: `tgpt` CLI는 ChatGPT 계정 로그인을 지원하지 않으므로 사용하지 않는다.

## Antigravity CLI (Gemini CLI의 공식 후속)

    curl -fsSL https://antigravity.google/cli/install.sh | bash

`~/.local/bin/agy`에 설치된다.

**로그인 (2026-10-15 검증됨):** VM 헤드리스 환경에서는 `agy`의 TUI OAuth 화면을 캡처할 수 없으므로 아래 방식을 사용한다.

1. 맥 터미널에서 `agy` 실행 → 화면의 Google 로그인 URL을 브라우저에서 열고 승인
2. `~/.gemini/antigravity-cli/antigravity-oauth-token` 파일을 VM에 전달 (Google Drive 공유 또는 파일 첨부)
3. VM의 `~/.gemini/antigravity-cli/`에 배치, 권한 600 설정

**헤드리스 호출 설정:** `~/.bashrc`에 아래 래퍼를 등록한다 (권한 프롬프트 우회).

    agy() { command agy --dangerously-skip-permissions "$@"; }

확인:

    agy --version
    agy -p "한 단어로 답해: 테스트"

정상 응답이 오면 완료. 호출은 `agy -p "프롬프트"` 형식으로 한다.

---

# 3. VM 교체 후 자동 유지 프롬프트

아래 코드 블록의 복사 버튼을 눌러 전체를 복사한 뒤 Muse에게 붙여넣으면 된다. 한 번만 실행하면 된다.

```
VM이 교체돼도 오케스트레이션 세팅이 유지되도록 설정해줘.

[1] 복원 스크립트 확인
- ~/workspace/toolchain/restore.sh 파일이 있는지 확인
- 없으면 아래 스크립트를 그대로 만들어줘:
  #!/bin/bash
  RESTORE_LOG="/tmp/toolchain-restore.log"
  log() { echo "[$(date '+%H:%M:%S')] $1" | tee -a "$RESTORE_LOG"; }
  command -v claude >/dev/null 2>&1 || { log "claude 없음 -> 설치"; npm i -g @anthropic-ai/claude-code 2>&1 | tail -2 | tee -a "$RESTORE_LOG"; }
  command -v agy >/dev/null 2>&1 || { log "agy 없음 -> 설치"; curl -fsSL https://antigravity.google/cli/install.sh | bash 2>&1 | tail -3 | tee -a "$RESTORE_LOG"; }
  python3 -c "import pproxy" 2>/dev/null || pip install --break-system-packages -q pproxy
  if ! (exec 3<>/dev/tcp/127.0.0.1/3129) 2>/dev/null; then
    nohup /usr/local/bin/pproxy -l http://127.0.0.1:3129 -r http://hatch-egress-proxy:3128 >/tmp/pproxy.log 2>&1 &
  fi
- chmod +x ~/workspace/toolchain/restore.sh로 실행 권한 부여

[2] ~/.bashrc에 복원 훅 추가
~/.bashrc 맨 아래에 아래 내용이 없으면 추가해줘:
  # VM 교체 후 툴체인 자동 복원 (muse-orchestration)
  if [ -f "$HOME/workspace/toolchain/restore.sh" ]; then
    if ! command -v claude >/dev/null 2>&1 || ! command -v agy >/dev/null 2>&1; then
      bash "$HOME/workspace/toolchain/restore.sh" >/tmp/toolchain-restore.log 2>&1 &
    fi
  fi
주의: 비대화형 셸에서는 ~/.bashrc가 실행되지 않을 수 있다. 이 경우 세션 시작 시 수동으로 bash ~/workspace/toolchain/restore.sh를 실행한다.

[3] ~/.bashrc에 ego light 환경변수 확인
아래가 없으면 추가해줘:
  export PATH="$HOME/.local/bin:$PATH"
  export EGO_HEADLESS=1
  export EGO_CHROME_PATH=$HOME/.local/bin/chrome-no-sandbox

[4] 최종 보고
- restore.sh: 있음 / 새로 만듦
- .bashrc 복원 훅: 있음 / 추가함
- .bashrc ego light 환경변수: 있음 / 추가함
```

---

# 4. 역할 분담 (오케스트레이션 기본값)

설치가 끝나면 아래 역할 분담이 모든 업무의 기본값이다.

| 역할 | 담당 | 하는 일 |
|------|------|---------|
| 두뇌 (전략·판단) | GPT (Codex CLI) | 계획 수립, 깊은 분석, 아키텍처 결정 |
| 두뇌 (코드) | Claude (Claude Code) | 코드 작성·수정·리뷰 |
| 리서치 | Antigravity (agy CLI) | 웹 리서치, 자료 조사 |
| 손발·창구 | Muse | 도구 실행, 파일 작업, 사용자와 소통 |

호출 방법:

    # GPT에게 깊은 사고 위임
    echo "프롬프트" | codex exec --skip-git-repo-check -

    # Claude에게 코드 작업 위임
    claude -p "프롬프트"

    # Antigravity에게 리서치 위임
    agy -p "프롬프트"

라우팅 규칙:
- 단순 채팅·실행 → Muse가 직접 처리
- 깊은 사고·판단 → GPT에게 위임
- 코드 작업 → Claude에게 위임
- 리서치 → Antigravity에게 위임
- 브라우저 작업 → ego light로 Muse가 직접 처리 (API/CLI로 안 될 때만)

브라우저 사용 원칙: **API/CLI로 되면 그걸 먼저 쓴다.** ego light는 로그인 필요 사이트, JS 렌더링 페이지, 스크린샷이 필요할 때만 사용한다.

---

# 5. 설치 후 검증

    which claude codex agy
    claude auth status --text     # Login method 확인
    codex login status            # Logged in using ChatGPT 확인
    agy -p "한 단어로 답해: 테스트"  # 정상 응답 확인
    ego-browser --doctor          # 데몬·크롬 상태 확인
    ss -tln | grep 3129           # pproxy 실행 확인

---

## 참고 문서

- [README.md](README.md) — 프로젝트 소개
- [ORCHESTRATION.md](ORCHESTRATION.md) — 역할 분담 상세, 라우팅 규칙, 워크플로우
- [INSTALL.md](INSTALL.md) — 상세 설치 가이드
- [BASICSETTING.md](BASICSETTING.md) — VM 전역 소프트웨어 명세
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — 문제 해결
