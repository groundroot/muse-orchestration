# 설치 가이드

Muse 오케스트레이션에 필요한 도구들의 설치 방법과 주의사항이다.

## 필요한 도구

| 도구 | 용도 | 설치 |
|------|------|------|
| Muse | 손발·창구 | Muse 앱 설치 |
| Codex CLI | GPT 두뇌 (깊은 사고) | `npm i -g @openai/codex` |
| Claude Code | Claude 두뇌 (코드 작업) | `npm i -g @anthropic-ai/claude-code` |

## Codex CLI 설치

```bash
npm i -g @openai/codex
```

로그인:

```bash
codex login
```

ChatGPT 계정으로 OAuth 로그인을 하면 된다. device-auth 방식도 지원한다.

로그인 확인:

```bash
codex login status
```

정상 로그인되면 `Logged in using ChatGPT`라고 표시된다.

비대화형 실행:

```bash
echo "프롬프트" | codex exec --skip-git-repo-check -
```

### 주의사항

- `tgpt` CLI는 ChatGPT 계정 로그인을 지원하지 않는다. Codex CLI를 쓸 것.
- Codex에서 특정 모델(예: GPT-6.1 Sol)이 `400 invalid_request_error`로 실패하면 해당 계정·버전에서 미지원인 것이다. 기본 모델로 진행한다.
- 인증 파일 `~/.codex/auth.json`은 절대 채팅이나 문서에 붙여넣지 않는다.

## Claude Code 설치

```bash
npm i -g @anthropic-ai/claude-code
```

로그인:

```bash
claude auth login
```

1번(Claude 구독 계정)을 선택하면 인증 URL이 표시된다. 브라우저가 있는 기기에서 URL을 열고 Authorize를 클릭한 뒤, 받은 승인 코드를 터미널에 붙여넣는다.

로그인 확인:

```bash
claude auth status --text
```

`Login method: Claude Team account` (또는 Pro/Max 계정)로 표시되면 정상이다.

### 주의사항

- **VM·클라우드 공유 IP 환경에서는 OAuth 토큰 교환 단계에서 HTTP 429가 발생할 수 있다.** 이 경우 로그인이 완료되지 않는다. 해결법은 [TROUBLESHOOTING.md](TROUBLESHOOTING.md#1-claude-code-oauth-로그인-시-http-429)를 참조.
- 승인 코드·access token·refresh token은 1회용이다. 실패하면 처음부터 다시 진행한다.
- 인증 파일 `~/.claude/.credentials.json`은 권한 600으로 유지한다 (`chmod 600`).
- 인증 파일은 파일 형태로만 전달한다. 채팅·메신저에 직접 붙여넣지 않는다.

## VM 환경 주의사항

Muse의 VM(리눅스)은 교체될 수 있다. 교체되면 아래가 초기화된다.

- npm 전역 설치 (`codex`, `claude` 바이너리)
- 실행 중인 프로세스 (pproxy, ego-browser 데몬 등)

유지되는 것:

- `~/workspace` 전체
- `~/.claude/.credentials.json` (Claude 인증 유지됨)
- `~/.codex/auth.json` (Codex 인증 유지됨)

VM 교체 후 복원 순서:

1. `npm i -g @anthropic-ai/claude-code @openai/codex` 재설치
2. `claude auth status --text`로 로그인 유지 확인 (재인증 불필요)
3. `codex login status`로 로그인 유지 확인
4. 브라우저 작업이 필요하면 pproxy 시작:
   ```bash
   nohup /usr/local/bin/pproxy -l http://127.0.0.1:3129 -r http://hatch-egress-proxy:3128 >/tmp/pproxy.log 2>&1 &
   ```

재설치 목록은 `~/workspace/toolchain/MANIFEST.md`에 기록해 두면 복원이 빠르다.
