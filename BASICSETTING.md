# Basic Setting — VM 전역 소프트웨어 자동 복원

VM이 교체되면 `/usr/lib/node_modules` 등 전역 설치가 초기화된다.
이 문서의 명세대로 소프트웨어를 즉시 재설치하면 수동 재설치가 필요 없다.
홈 디렉토리(`~`)는 유지되므로 인증은 다시 하지 않는다.

## 최우선 규칙

**어떤 작업보다 먼저 툴체인 상태를 확인한다.**
`claude`, `agy` 바이너리가 없으면 아래 명세대로 즉시 재설치한다.
VM 교체 시 전역 설치가 날아가므로, 세션 시작 시 이 확인이 기본값이다.

```bash
command -v claude >/dev/null || npm i -g @anthropic-ai/claude-code
command -v agy >/dev/null || curl -fsSL https://antigravity.google/cli/install.sh | bash
```

## npm 전역 패키지

| 패키지 | 명령 | 용도 | 설치 명령 |
|--------|------|------|-----------|
| @anthropic-ai/claude-code | `claude` | Claude 두뇌 (코드 작업) | `npm i -g @anthropic-ai/claude-code` |

## 직접 설치 (npm 아님)

| 도구 | 명령 | 용도 | 설치 명령 |
|------|------|------|-----------|
| Antigravity CLI | `agy` | 리서치 (Gemini CLI 후속) | `curl -fsSL https://antigravity.google/cli/install.sh \| bash` |

## pip 패키지

| 패키지 | 용도 | 설치 명령 |
|--------|------|-----------|
| pproxy | 로컬 포워드 프록시 (Chrome → egress 프록시 체이닝) | `pip install --break-system-packages pproxy` |

## 이미지 내장 (재설치 불필요)

| 도구 | 경로 | 비고 |
|------|------|------|
| codex | `/opt/hatch-image/bin/codex` | Codex CLI, VM 이미지에 포함 |

## 자동 복원 스크립트

아래 스크립트를 `~/workspace/toolchain/restore.sh`로 저장하면 check-first 방식으로 복원한다 (없으면 설치, 있으면 건너뜀).

```bash
#!/bin/bash
# VM 교체 후 전역 소프트웨어 자동 복원

RESTORE_LOG="/tmp/toolchain-restore.log"
log() { echo "[$(date '+%H:%M:%S')] $1" | tee -a "$RESTORE_LOG"; }

command -v claude >/dev/null 2>&1 || {
  log "claude 없음 → 설치"
  npm i -g @anthropic-ai/claude-code 2>&1 | tail -2 | tee -a "$RESTORE_LOG"
}

command -v agy >/dev/null 2>&1 || {
  log "agy 없음 → 설치"
  curl -fsSL https://antigravity.google/cli/install.sh | bash 2>&1 | tail -3 | tee -a "$RESTORE_LOG"
}

python3 -c "import pproxy" 2>/dev/null || {
  log "pproxy 없음 → 설치"
  pip install --break-system-packages -q pproxy 2>&1 | tail -1 | tee -a "$RESTORE_LOG"
}

if ! (exec 3<>/dev/tcp/127.0.0.1/3129) 2>/dev/null; then
  log "pproxy 실행 → 시작"
  nohup /usr/local/bin/pproxy -l http://127.0.0.1:3129 -r http://hatch-egress-proxy:3128 >/tmp/pproxy.log 2>&1 &
fi

log "복원 확인:"
command -v claude >/dev/null && log "  claude $(claude --version 2>/dev/null)"
command -v agy >/dev/null && log "  agy $(agy --version 2>/dev/null)"
command -v codex >/dev/null && log "  codex $(codex --version 2>/dev/null)"
```

실행:

```bash
bash ~/workspace/toolchain/restore.sh
```

## 복원 후 확인

```bash
claude --version && claude auth status --text   # Login method 확인
codex --version && codex login status           # Logged in using ChatGPT 확인
agy --version && agy -p "test"                  # 정상 응답 확인
ss -tln | grep 3129 || echo "pproxy not running"
```

## 인증 (재로그인 불필요)

아래 파일은 홈 디렉토리에 있어 VM 교체에도 유지된다. 파일이 있으면 재로그인하지 않는다.

- `~/.claude/.credentials.json` — Claude 계정
- `~/.codex/auth.json` — ChatGPT 계정
- `~/.gemini/antigravity-cli/antigravity-oauth-token` — Antigravity 인증 (맥에서 로그인 후 토큰 파일 이전, 2026-10-15 검증)

## 참고

- 상세 설치법은 [INSTALL.md](INSTALL.md)를 참조
- 문제 해결은 [TROUBLESHOOTING.md](TROUBLESHOOTING.md)를 참조
