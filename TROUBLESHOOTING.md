# 트러블슈팅

셋업 과정에서 실제로 발생한 문제들과 해결 방법이다.

## 1. Claude Code OAuth 로그인 시 HTTP 429

### 증상

`claude auth login` 후 승인 코드를 입력하면 아래 오류로 실패한다.

```json
{"error": {"type": "rate_limit_error", "message": "Rate limited. Please try again later."}}
```

### 원인

`platform.claude.com`의 `/v1/oauth/token` 엔드포인트가 VM의 공유 외부 IP를 제한한 것이다. 확인된 사실:

- 같은 계정으로 개인 네트워크의 맥에서는 로그인 성공 → 계정 문제 아님
- 같은 VM에서 `api.anthropic.com`은 정상 응답 → IP 전체 차단 아님
- 429가 8시간 이상 지속 → 일시적 제한 아님

즉, **OAuth 토큰 엔드포인트 + 공유 IP 조합에 걸린 IP 기반 제한**이다. VM의 모든 외부 통신이 단일 공유 프록시를 경유하므로 우회가 불가능하다.

### 해결법

다른 네트워크의 기기(맥·PC)에서 먼저 로그인한 뒤, 인증 파일을 옮긴다.

**맥에서:**

```bash
claude auth login
# 1번 선택 → 브라우저에서 Authorize → 승인 코드 입력
claude auth status --text
# "Login method: Claude Team account" 확인
```

**인증 파일 이동:**

맥의 `~/.claude/.credentials.json`을 VM의 `~/.claude/.credentials.json`에 복사한다.

```bash
# VM에서
mkdir -p ~/.claude
cp [전달받은 파일] ~/.claude/.credentials.json
chmod 600 ~/.claude/.credentials.json
claude auth status --text
```

파일에는 토큰이 들어 있으니 파일 형태로만 전달한다.

## 2. npm 설치 시 EEXIST 오류

### 증상

```bash
npm i -g @anthropic-ai/claude-code
# npm error EEXIST: file already exists
```

### 원인

이미 설치되어 있는데 다시 설치하려고 해서 발생한다.

### 해결법

설치가 필요 없는 경우이므로 건너뛴다. 버전만 확인한다.

```bash
claude --version
```

정말 재설치가 필요하면 `--force` 옵션을 쓴다.

## 3. VM 교체 후 CLI가 사라짐

### 증상

`claude: command not found` 또는 `codex: command not found`

### 원인

VM 교체 시 npm 전역 설치가 초기화된다. 인증 파일(`~/.claude/`, `~/.codex/`)과 `~/workspace`는 유지된다.

### 해결법

재설치만 하면 된다. 로그인은 유지되므로 다시 로그인할 필요 없다.

```bash
npm i -g @anthropic-ai/claude-code @openai/codex
claude auth status --text   # 로그인 유지 확인
codex login status          # 로그인 유지 확인
```

## 4. ego-browser (Chrome)에서 페이지 접속 실패

### 증상

`browser.openOrReuseTab(url)` 후 `chrome-error://chromewebdata/`가 표시된다. `ERR_TUNNEL_CONNECTION_FAILED` 또는 인증서 오류(Privacy error).

### 원인

두 가지가 겹쳐 있었다.

1. 크롬의 직접 CONNECT 요청이 egress 프록시에서 거부됨 (curl·파이썬은 정상)
2. 프록시가 TLS 중간자 검사를 수행하는데, 그 CA("Hatch Sandbox Egress CA")를 크롬이 신뢰하지 않음

### 해결법

로컬 포워드 프록시(`pproxy`)를 중간에 둔다.

```bash
# pproxy 설치 (1회)
pip install --break-system-packages pproxy

# pproxy 시작
nohup /usr/local/bin/pproxy -l http://127.0.0.1:3129 -r http://hatch-egress-proxy:3128 >/tmp/pproxy.log 2>&1 &
```

크롬 래퍼(`~/.local/bin/chrome-no-sandbox`)를 아래처럼 설정한다.

```sh
#!/bin/sh
# pproxy 자동 시작
if ! (exec 3<>/dev/tcp/127.0.0.1/3129) 2>/dev/null; then
  nohup /usr/local/bin/pproxy -l http://127.0.0.1:3129 -r http://hatch-egress-proxy:3128 >/tmp/pproxy.log 2>&1 &
  for i in 1 2 3 4 5 6 7 8 9 10; do
    (exec 3<>/dev/tcp/127.0.0.1/3129) 2>/dev/null && break
    sleep 0.5
  done
fi
exec /opt/meta-chromium/chrome --no-sandbox --disable-dev-shm-usage \
  --proxy-server="http://127.0.0.1:3129" \
  --proxy-bypass-list="$(echo "${no_proxy:-$NO_PROXY}" | tr ',' ';')" \
  --ignore-certificate-errors \
  "$@"
```

데몬 재시작 후 `https://example.com` 접속으로 확인한다.

참고: 프록시 정책상 차단된 도메인(`clients2.google.com`, `redirector.gvt1.com` 등)은 이 방법으로도 접속 불가하다.

## 5. Codex에서 특정 모델 미지원

### 증상

```
400 invalid_request_error
```

특정 모델(예: GPT-6.1 Sol) 지정 시 발생한다.

### 원인

해당 Codex 버전·계정에서 미지원 모델이다.

### 해결법

기본 모델로 진행한다. 모델 지정 없이 `codex exec`을 쓰면 된다.

## 6. Chrome 데몬이 죽어 있음

### 증상

`ego-browser --doctor`는 정상이라고 보고하는데 실제로는 프로세스가 없다.

### 원인

`--doctor`가 오래된 캐시 상태를 보고하는 경우가 있다.

### 해결법

프로세스 직접 확인 후 재시작한다.

```bash
pgrep -f "remote-debugging-port=9222" || echo "dead"
rm -f ~/.local/share/ego-lite/host.sock
ego-browser --doctor
```
