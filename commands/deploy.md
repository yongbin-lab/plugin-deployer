---
description: 플러그인을 GitHub 마켓플레이스에 배포합니다. 사용법 — /plugin-deployer:deploy <플러그인 경로>
---

# Plugin Deployer

사용자가 배포하려는 플러그인: $ARGUMENTS

아래 워크플로우를 순서대로 따르세요. 각 단계에서 문제가 생기면 멈추고 사용자에게 알려주세요.

---

## 1단계: 플러그인 검증

주어진 경로에서 플러그인 구조를 확인합니다:

1. `.claude-plugin/plugin.json`이 존재하는지 확인
2. `plugin.json`에서 `name`, `description`, `version` 필드 확인
3. 컴포넌트 디렉토리(commands/, skills/, agents/, hooks/)가 `.claude-plugin/` 바깥 플러그인 루트에 있는지 확인
4. 문제가 있으면 사용자에게 알리고 수정 제안

검증 통과 시 플러그인 정보를 요약해서 보여주세요:
- 이름, 설명, 버전
- 포함된 컴포넌트 목록

---

## 2단계: 배포 정보 수집

사용자에게 필요한 정보를 확인합니다:

1. **GitHub 유저네임** — `gh api user --jq '.login'`으로 자동 감지 시도. 실패하면 질문
2. **레포 이름** — 기본값: 플러그인 이름과 동일. 변경 원하는지 확인
3. **마켓플레이스 이름** — 기본값: `<유저네임>-plugins`. 변경 원하는지 확인
4. **공개/비공개** — 기본값: public
5. **라이선스** — 기본값: MIT

간단히 기본값을 보여주고 "이대로 진행할까요?" 한 번만 물어보세요. 질문을 여러 번 나눠서 하지 마세요.

---

## 3단계: 마켓플레이스 구조 생성

플러그인을 감싸는 마켓플레이스 레포 구조를 만듭니다:

```
<레포-이름>/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── <플러그인-이름>/        ← 원본 플러그인을 여기에 복사
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       ├── skills/
│       └── ...
└── README.md
```

### marketplace.json 생성:
```json
{
  "name": "<마켓플레이스-이름>",
  "owner": {
    "name": "<GitHub 유저네임>"
  },
  "metadata": {
    "description": "<마켓플레이스 설명>",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "<플러그인-이름>",
      "source": "./<플러그인-이름>",
      "description": "<플러그인 설명>",
      "version": "<플러그인 버전>"
    }
  ]
}
```

### README.md 생성:
플러그인 이름, 설명, 설치 방법, 사용법을 포함하는 README를 작성하세요.

설치 방법 섹션에는 반드시 다음을 포함:
```
/plugin marketplace add <유저네임>/<레포-이름>
/plugin install <플러그인-이름>@<마켓플레이스-이름>
```

---

## 4단계: GitHub에 배포

순서대로 실행합니다:

1. `gh auth status`로 GitHub 인증 상태 확인. 인증 안 되어 있으면 `gh auth login` 안내
2. 마켓플레이스 디렉토리에서 `git init` + 초기 커밋
3. `gh repo create <레포-이름> --public --source=. --push`로 레포 생성 및 푸시
   - 비공개 선택 시 `--private` 사용
4. 성공하면 결과 URL 표시

---

## 5단계: 완료 안내

배포 완료 후 사용자에게 보여줄 내용:

```
✅ 배포 완료!

📦 레포: https://github.com/<유저네임>/<레포-이름>

🔧 다른 사람이 설치하려면:
  /plugin marketplace add <유저네임>/<레포-이름>
  /plugin install <플러그인-이름>@<마켓플레이스-이름>

📝 플러그인 업데이트 시:
  plugins/<플러그인-이름>/ 수정 후 git push
  사용자는 /plugin marketplace update 로 업데이트
```

---

## 중요 규칙

- `gh` CLI가 없으면 설치 방법을 안내하세요 (brew install gh)
- git config가 안 되어 있으면 설정을 도와주세요
- 이미 같은 이름의 GitHub 레포가 있으면 사용자에게 확인
- 기존 마켓플레이스 레포에 플러그인을 추가하는 경우도 처리하세요 (marketplace.json의 plugins 배열에 추가)
- 절대 `--force` 푸시하지 마세요
- 민감한 파일(.env, credentials 등)이 있으면 경고하세요
