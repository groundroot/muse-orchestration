# Easy Install — Muse에게 붙여넣는 설치 프롬프트

새 VM에서 아래 프롬프트를 그대로 복사해 Muse에게 보내면 도구 설치가 자동으로 진행된다.
로그인은 사용자가 직접 하는 단계로 분리되어 있다.

---

```
Muse 오케스트레이션 도구를 설치해줘. 아래 순서대로 진행하고, 각 단계마다 결과를 보고해줘.

## 1. 사전 확인
- `node --version`으로 Node.js 18 이상인지 확인
- `which claude codex gemini`로 이미 설치된 게 있는지 확인
- 이미 설치된 건 건너뛰고, 없는 것만 설치해

## 2. Claude Code 설치
```bash
npm i -g @anthropic-ai/claude-code
```
설치 후 `claude --version`으로 확인해줘.

## 3. Codex CLI 설치
```bash
npm i -g @openai/codex
```
단, `/opt/hatch-image/bin/codex`에 이미 있으면 npm 설치를 건너뛰고 그걸 사용해.
설치 후 `codex --version`으로 확인해줘.

## 4. Gemini CLI 설치
```bash
npm i -g @google/gemini-cli
```
설치 후 `gemini --version`으로 확인해줘.

## 5. pproxy 설치 (브라우저용)
```bash
pip install --break-system-packages pproxy
```
설치 후 pproxy가 127.0.0.1:3129에서 실행 중인지 확인하고, 안 떠 있으면 시작해줘:
```bash
nohup /usr/local/bin/pproxy -l http://127.0.0.1:3129 -r http://hatch-egress-proxy:3128 >/tmp/pproxy.log 2>&1 &
```

## 6. 최종 보고
아래 형식으로 결과만 간단히 보고해줘:
- claude: 버전 / 설치됨 또는 이미 있었음
- codex: 버전 / 설치됨 또는 이미 있었음
- gemini: 버전 / 설치됨 또는 이미 있었음
- pproxy: 실행 중 또는 시작함

로그인은 내가 직접 할 테니 설치만 하고 멈춰줘.
```

---

## 로그인 (설치 후 사용자가 직접)

### Claude Code

```bash
claude auth login
```

1번(Claude 구독 계정)을 선택하면 인증 URL이 표시된다. 브라우저가 있는 기기에서 URL을 열고 Authorize를 클릭한 뒤, 받은 승인 코드를 터미널에 붙여넣는다.

확인: `claude auth status --text` → `Login method`가 표시되면 정상.

주의: VM 공유 IP에서 HTTP 429가 나면 개인 네트워크 기기에서 로그인 후 `~/.claude/.credentials.json`을 파일로 옮긴다.

### Codex CLI

```bash
codex login
```

ChatGPT 계정으로 OAuth 로그인. 확인: `codex login status` → `Logged in using ChatGPT`.

### Gemini CLI

**API 키 방식 (헤드리스 권장):**

1. https://aistudio.google.com/apikey 에서 "Create API key"
2. `export GEMINI_API_KEY="키"` 후 `~/.bashrc`에 영구 추가

확인: `gemini -p "한 단어로 답해: 테스트"` → 정상 응답이면 완료.

## 참고

- 상세 설치법: [INSTALL.md](INSTALL.md)
- VM 교체 후 자동 복원: [BASICSETTING.md](BASICSETTING.md)
- 문제 해결: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
