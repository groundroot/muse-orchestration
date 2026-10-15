# Muse 오케스트레이션

Claude와 GPT를 두뇌로, Muse(알프레드)를 손발로 쓰는 AI 협업 셋업이다.

## 역할 분담

| 역할 | 담당 |
|------|------|
| 두뇌 (깊은 사고·판단) | GPT 6 — Codex CLI |
| 두뇌 (코드 작업) | Claude — Claude Code |
| 손발 (실행) | Muse (알프레드) |
| 창구 (대화) | Muse (알프레드) |

- 단순 채팅·실행은 Muse가 직접 처리한다.
- 깊이 있는 사고·판단이 필요하면 GPT 6에게 위임한다.
- 코드 관련 작업은 Claude에게 위임한다.

## 문서

- [ORCHESTRATION.md](ORCHESTRATION.md) — 오케스트레이션 세팅 프롬프트 (역할 정의·라우팅 규칙·워크플로우)
- [INSTALL.md](INSTALL.md) — 설치 가이드 및 주의사항
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — 셋업 시 발생한 문제와 해결법

## 빠른 시작

1. [INSTALL.md](INSTALL.md)를 보고 Codex CLI와 Claude Code를 설치·로그인한다.
2. [ORCHESTRATION.md](ORCHESTRATION.md)의 라우팅 규칙대로 업무를 분기한다.
3. 막히면 [TROUBLESHOOTING.md](TROUBLESHOOTING.md)를 확인한다.
