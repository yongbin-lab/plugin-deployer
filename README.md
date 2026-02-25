# plugin-deployer

> "배포해줘"라고 말하면 Claude Code 플러그인을 GitHub 마켓플레이스에 바로 배포해주는 플러그인

## 설치

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add yongbin-lab/plugin-deployer

# 2. 플러그인 설치
/plugin install plugin-deployer@yongbin-lab-plugins
```

또는 로컬에서 바로 테스트:

```bash
claude --plugin-dir ./plugin-deployer
```

## 사용법

### 자동 트리거 (Skill)

Claude에게 자연스럽게 말하면 자동으로 배포를 시작합니다:

- "이 플러그인 배포해줘"
- "마켓플레이스에 올려줘"
- "플러그인 공유하고 싶어"
- "deploy this plugin to marketplace"

### 직접 호출 (Command)

```bash
/plugin-deployer:deploy <플러그인 경로>
```

예시:
```bash
/plugin-deployer:deploy ./my-awesome-plugin
```

## 동작 흐름

1. 플러그인 구조 검증 (plugin.json, 디렉토리 구조)
2. 배포 설정 자동 감지 + 확인 (GitHub 유저네임, 레포명, 공개/비공개)
3. 마켓플레이스 구조 생성 (marketplace.json + README + 플러그인 복사)
4. GitHub 레포 생성 및 푸시
5. 설치 명령어 안내

## 전제 조건

- [gh CLI](https://cli.github.com/) 설치 및 인증 (`gh auth login`)
- Git 설정 완료

## 플러그인 구조

```
plugin-deployer/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   └── deploy.md
└── skills/
    └── deploy-plugin/
        ├── SKILL.md
        └── references/
            └── marketplace-structure.md
```

## 라이선스

MIT
