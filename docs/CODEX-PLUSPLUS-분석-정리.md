# Codex++ 전수조사 & 활용 전략 정리 📘

> 작성일: 2026-09-20
> 작성: 카리나 (Claude Code) 💖
> 대상 저장소: 이 저장소(`codex-plusplus`) 전체 코드/문서 전수조사 결과

## 🔗 관련 GitHub 주소

| 항목 | 주소 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/codex-plusplus |
| **원본 저장소 (upstream)** | https://github.com/b-nnett/codex-plusplus |
| 이슈 트래커 | https://github.com/b-nnett/codex-plusplus/issues |
| 트윅 스토어 레지스트리 | https://b-nnett.github.io/codex-plusplus/store/index.json |
| Discord 커뮤니티 | https://discord.gg/6bY6gGX36H |
| 설치 스크립트 (macOS/Linux) | https://raw.githubusercontent.com/b-nnett/codex-plusplus/main/install.sh |
| 설치 스크립트 (Windows) | https://raw.githubusercontent.com/b-nnett/codex-plusplus/main/install.ps1 |

---

## 목차

1. [한 줄 요약](#1-한-줄-요약)
2. [프로젝트 개요](#2-프로젝트-개요)
3. [폴더 구조 전수조사](#3-폴더-구조-전수조사)
4. [동작 원리](#4-동작-원리)
5. [Tweak 시스템](#5-tweak-시스템)
6. [Tweak Store](#6-tweak-store)
7. [MCP 연동](#7-mcp-연동)
8. [네이티브 브릿지](#8-네이티브-브릿지-macos)
9. [설치 및 사용법](#9-설치-및-사용법)
10. [플러그인 vs 스킬 vs MCP](#10-플러그인-vs-스킬-vs-mcp)
11. [API 토큰 필요 여부](#11-api-토큰-필요-여부)
12. [GitHub에서 유명한 이유](#12-github에서-유명한-이유)
13. [로컬 에이전트 구축 활용성](#13-로컬-에이전트-구축-활용성)
14. [React / PHP 로 만들 수 있나](#14-react--php-로-만들-수-있나)
15. [수익화 아이디어](#15-수익화-아이디어)
16. [실행 로드맵](#16-실행-로드맵)
17. [리스크 및 주의사항](#17-리스크-및-주의사항)

---

## 1. 한 줄 요약

> **OpenAI Codex 데스크톱 앱을 로컬에서 패치해서, 커뮤니티가 만든 확장 기능(Tweak)을 설치·실행할 수 있게 해주는 비공식 플러그인 시스템.**

BetterDiscord(Discord), Spicetify(Spotify)와 같은 장르의 프로젝트다.

- 버전: `1.0.0`
- 라이선스: MIT
- 총 코드량: 약 35,900줄
- 지원 OS: macOS / Windows / Linux
- **OpenAI 공식 프로젝트가 아님** (비공식, 사용자 책임)

---

## 2. 프로젝트 개요

### 제공하는 것

- 로컬 `tweaks/` 폴더
- 렌더러 / 메인 프로세스 트윅을 로드하는 런타임
- Codex 설정 화면 안의 "Codex++" 섹션
- install / repair / update / debug / doctor CLI
- Codex 업데이트 후 자동 복구 워처(watcher)
- 트윅 작성자용 공개 SDK
- macOS 고급 트윅용 네이티브 브릿지 API

### 제공하지 않는 것 (README 명시)

> "It does not replace Codex, proxy your account, or run a separate Codex clone."

- Codex를 대체하지 않음
- 계정을 프록시하지 않음
- 별도 Codex 클론을 실행하지 않음

---

## 3. 폴더 구조 전수조사

```
codex-plusplus/
├── packages/                 # npm workspaces (4+1개)
│   ├── loader/               # 88줄. app.asar에 주입되는 최소 스텁
│   │   └── loader.cjs
│   ├── runtime/              # 두뇌. 트윅 발견/생명주기/설정 UI 주입
│   │   └── src/
│   │       ├── main.ts                 # 메인 프로세스 부트스트랩
│   │       ├── tweak-discovery.ts      # 트윅 탐색
│   │       ├── tweak-lifecycle.ts      # start/stop 관리
│   │       ├── tweak-store.ts          # 스토어 클라이언트
│   │       ├── mcp-sync.ts             # config.toml MCP 동기화
│   │       ├── native-bridge.ts        # 네이티브 호출 브릿지
│   │       ├── native-paths.ts         # 경로 탈출 방지 검증
│   │       ├── codex-runtime-probe.ts  # Owl/Electron 판별
│   │       ├── watcher-health.ts       # 워처 상태 점검
│   │       ├── browser-ui.ts           # 브라우저 호스트 모드
│   │       ├── storage.ts / logging.ts
│   │       └── preload/
│   │           ├── index.ts
│   │           ├── react-hook.ts         # React DevTools 형태 훅
│   │           ├── settings-injector.ts  # Settings 다이얼로그 주입
│   │           ├── tweak-host.ts
│   │           └── manager.ts
│   ├── installer/            # CLI (codexplusplus 명령어)
│   │   └── src/
│   │       ├── cli.ts
│   │       ├── asar.ts          # asar 언팩/패치/리팩
│   │       ├── codesign.ts      # macOS 재서명
│   │       ├── integrity.ts     # Info.plist 해시 재계산
│   │       ├── fuses.ts         # Electron fuse 조작
│   │       ├── watcher.ts       # launchd/systemd/작업스케줄러
│   │       ├── plist.ts / paths.ts / state.ts / platform.ts
│   │       ├── ownership.ts / windows-cleanup.ts
│   │       ├── codex-window-services.ts
│   │       └── commands/        # install, repair, doctor, uninstall,
│   │                            # status, debug, safe-mode, update-codex,
│   │                            # self-update, create-tweak,
│   │                            # validate-tweak, dev-tweak, browser-ui
│   ├── sdk/                  # 635줄. 공개 타입 + validateTweakManifest
│   └── native-host/          # macOS Objective-C++ AppKit/Metal 호스트
├── docs/                     # 문서 15개
│   ├── ARCHITECTURE.md
│   ├── OWL-RUNTIME.md / OWL-BRIDGE-ROADMAP.md
│   ├── TROUBLESHOOTING.md / WRITING-TWEAKS.md
│   ├── releases/             # 0.1.2 ~ 0.1.7 릴리스 노트
│   └── tweaks/               # api-reference, manifest, mcp,
│                             # native-bridge, runtime-lifecycle,
│                             # ui-and-dom, typescript-and-bundling,
│                             # getting-started, distribution-debugging
├── store/
│   ├── index.json            # 승인된 트윅 14개 레지스트리
│   └── icons/                # 트윅 아이콘 14개
├── tweaks/AGENTS.md          # AI 에이전트용 트윅 작성 가이드
├── Formula/codexplusplus.rb  # Homebrew 공식
├── bin/codexplusplus.js      # CLI 진입점 (없으면 자동 빌드)
├── install.sh / install.ps1  # 원클릭 설치
├── update.sh / update.ps1
├── .github/workflows/ci.yml
└── CLAUDE.md                 # 프로젝트 페르소나 설정
```

### 패키지별 역할 요약

| 패키지 | 역할 | 비유 |
|---|---|---|
| `loader` (88줄) | app.asar 안에 심어지는 최소 스텁 | 잠입 요원 |
| `runtime` | 트윅 관리/실행/UI 주입 | 본사 두뇌 |
| `installer` | 모든 CLI 명령어 | 설치기사 |
| `sdk` | 트윅 개발자용 타입/API | 사용설명서 |
| `native-host` | macOS AppKit/Metal | 특수부대 |

---

## 4. 동작 원리

### 4-1. Codex 앱의 정체

Codex 데스크톱은 Electron(현재 macOS 빌드는 OpenAI 자체 셸인 **Owl**) 기반이다.
즉 내부가 자바스크립트이므로 읽고 수정할 수 있다.

### 4-2. 설치 흐름

```
1. Codex 앱 위치를 찾는다
2. 패치 전 원본 파일들을 backup/ 에 백업
3. app.asar 안의 package.json "main" 을 로더로 교체
4. 런타임을 유저 데이터 디렉터리에 스테이징
5. 필요 시 앱을 재서명 (macOS Gatekeeper 통과)
6. 향후 Codex 업데이트 대비 워처 설치
```

**핵심 패치 내용 (약 1KB):**

```json
{
  "main": "codex-plusplus-loader.cjs",
  "__codexpp": {
    "originalMain": "index.js",
    "userRoot": "~/Library/Application Support/codex-plusplus"
  }
}
```

### 4-3. 로더 설계 철학

`packages/loader/loader.cjs` 주석 원문:

> **"broken tweak system > broken Codex"**
> (트윅 시스템이 깨지는 게 Codex가 깨지는 것보다 낫다)

```js
// 런타임 로드 실패해도 로그만 남기고 통과
safe("runtime", () => require(path.join(runtimeDir, "main.js")));

// 항상 원본 진입점으로 제어를 넘김
require("./" + originalMain);
```

즉 **Codex++가 완전히 망가져도 Codex 본체는 정상 실행된다.**

### 4-4. 부팅 시퀀스 (ARCHITECTURE.md 기준)

```
1. 사용자가 Codex.app 실행
2. macOS가 재서명된 서명 검증 → Gatekeeper 통과
3. Info.plist의 asar 무결성 해시 검사 → 새 해시와 일치 → 통과
4. package.json#main → codex-plusplus-loader.cjs 로드
5. 로더: userRoot 읽기 → 환경변수 설정 → runtime/main.js require
6. 런타임: 세션 API로 preload 등록(추가 방식) → 트윅 발견 → main 스코프 트윅 시작 → IPC 핸들러 등록
7. Codex가 BrowserWindow 생성 → Codex preload + Codex++ preload 둘 다 실행
8. preload: React 훅 설치 → IPC로 트윅 목록 수신 → renderer 트윅 start(api) → Settings 인젝터 시작
9. Settings 열면: [role="dialog"] 감지 → [role="tablist"]에 "Tweaks" 탭 추가 → 패널 표시
```

### 4-5. macOS 보안 우회 3단 콤보

| 검사 | 대응 |
|---|---|
| `Info.plist`의 `ElectronAsarIntegrity` 해시 | 패치된 asar 해시로 재계산 후 덮어씀 |
| 코드 서명 검증 | "Codex++ Local Signing" 로컬 인증서로 재서명 (`--no-local-signing`으로 ad-hoc 선택 가능) |
| Electron 퓨즈 | 구버전 Electron 빌드면 `EnableEmbeddedAsarIntegrityValidation` 비활성화 |

> **SIP(시스템 무결성 보호), 하드닝 런타임, 커널 보호는 절대 건드리지 않는다.** (문서 명시)

### 4-6. 설정 UI 주입 방식

React 소스를 패치하지 않고 **DOM 관찰(MutationObserver)** 로 끼어든다.

ARCHITECTURE.md 원문 요지:
- Codex는 Vite/Rollup 빌드 → 런타임에 모듈 레지스트리 노출 없음 → `webpackChunk` 트릭 불가
- 압축된 번들을 문자열 패치하면 릴리스마다 깨짐
- → 안정적인 affordance(Radix 속성, `[role="dialog"]`)에만 의존 → **Codex 업데이트돼도 대부분 그대로 동작**

### 4-7. 파일 저장 위치

**철학: Codex 앱 안에는 최소한만, 나머지는 전부 밖에**

| 항목 | 위치 |
|---|---|
| 로더 패치 (~1KB) | Codex `app.asar` 내부 |
| 런타임 | `<user-data>/runtime/` |
| 트윅 | `<user-data>/tweaks/` |
| 트윅 데이터(샌드박스) | `<user-data>/tweak-data/<tweak-id>/` |
| 설정 | `<user-data>/config.json` |
| 상태 | `<user-data>/state.json` |
| 로그 / 백업 | `<user-data>/log/`, `<user-data>/backup/` |

| OS | 기본 경로 |
|---|---|
| macOS | `~/Library/Application Support/codex-plusplus/` |
| Windows | `%APPDATA%/codex-plusplus/` |
| Linux | `$XDG_DATA_HOME/codex-plusplus/` 또는 `~/.local/share/codex-plusplus/` |

> Windows Store 설치본의 경우 `%LOCALAPPDATA%/codex-plusplus/store-apps/` 에 쓰기 가능한 관리 사본을 추가 생성한다.

**왜 밖에 두는가?** → 트윅을 고칠 때마다 115MB짜리 Codex asar를 다시 만들 필요가 없기 때문.

### 4-8. Codex 업데이트 대응

```
1. Sparkle이 새 Codex.app 다운로드 → 교체
2. 패치 사라짐 → 그래도 Codex는 정상 실행
3. 워처 발동 (macOS: launchd가 app.asar 감시 / Linux: systemd / Windows: 로그온 시)
4. `codex-plusplus repair --quiet` 실행
5. 멱등성: 해시가 그대로면 아무것도 안 함, 달라졌으면 새 앱에 재패치
```

트윅은 앱 밖에 있으므로 업데이트해도 사라지지 않는다.

또한 워처는 매시간 Codex++ 자체 업데이트도 확인한다
(`~/.codex-plusplus/source/packages/installer/dist/cli.js` 사용).
Settings → Codex Plus Plus → Config 에서 자동 업데이트를 끌 수 있다.

---

## 5. Tweak 시스템

### 5-1. 트윅 구조

```
my-tweak/
  manifest.json   # 필수 메타데이터
  index.js        # module.exports = { start(api), stop() }
  icon.png        # 선택
```

**최소 manifest.json:**

```json
{
  "id": "com.you.my-tweak",
  "name": "My Tweak",
  "version": "0.1.0",
  "githubRepo": "you/my-tweak",
  "description": "Adds a Codex++ settings page.",
  "scope": "renderer",
  "main": "index.js"
}
```

**최소 index.js:**

```js
module.exports = {
  start(api) {
    api.settings.registerPage({
      id: "main",
      title: api.manifest.name,
      render(root) {
        root.textContent = "Hello from Codex++.";
      },
    });
  },
  stop() {},
};
```

SDK 헬퍼 사용 시:

```js
const { defineTweak } = require("@codex-plusplus/sdk");
module.exports = defineTweak({
  start(api) { api.log.info("hello"); },
});
```

### 5-2. scope (실행 위치)

| scope | 실행 위치 | 가능한 작업 |
|---|---|---|
| `renderer` | Codex 화면(React UI) | DOM 조작, React Fiber 접근, 단축키 가로채기 |
| `main` | Electron 메인 프로세스 | 창 제어, 파일시스템, OS 기능, 네이티브 호출 |
| `both` | 둘 다 | 전부 |

### 5-3. permissions (13종)

`ipc`, `filesystem`, `network`, `settings`,
`codex-runtime`, `codex-windows`, `codex-views`, `codex-cdp`,
`codex.windows`, `codex.views`,
`native-module`, `native-view`, `native-helper`

### 5-4. TweakApi 표면 (SDK 기준)

```ts
interface TweakApi {
  manifest: Readonly<TweakManifest>;
  storage: TweakStorage;       // 키-값 저장
  log: TweakLogger;
  process: "renderer" | "main";
  settings?: SettingsApi;      // registerPage / registerSection
  react?: ReactApi;            // Fiber 트리 탐색
  ipc: TweakIpc;
  fs: TweakFs;                 // dataDir 샌드박스
  codex?: CodexApi;            // runtime / windows / views / cdp / native
}
```

`CodexApi` 주요 항목:

- `api.codex.runtime.getInfo()` — Owl/Electron 판별, 버전, 채널, 경로
- `api.codex.runtime.getCapabilities()` — 기능 지원 여부
- `api.codex.windows.*` — 창 열거/제어
- `api.codex.views.*` — WebContentsView 생성/부착
- `api.codex.cdp.*` — Chrome DevTools Protocol 타깃 접근
- `api.codex.native.*` — 네이티브 모듈/패널/헬퍼

> **주의: Owl 내부 API를 직접 쓰지 말고 반드시 SDK를 경유할 것.** (README 권고)

### 5-5. 개발 워크플로

```sh
codexplusplus create-tweak ./my-tweak --id com.you.my-tweak --name "My Tweak"
codexplusplus validate-tweak ./my-tweak
codexplusplus dev ./my-tweak
```

### 5-6. AGENTS.md 프라임 디렉티브

`tweaks/AGENTS.md` 는 **AI 코딩 에이전트가 읽고 따르도록** 쓰인 가이드다.

> **"Codex의 기존 UI 패턴을 따를 것. 새로운 시각 언어를 발명하지 말 것.
> 색상·크기·폰트를 하드코딩하지 말 것. Codex의 Tailwind 토큰
> (`text-token-*`, `bg-token-*`, `border-token-border`, `px-row-x`,
> `py-row-y`, `p-panel`, `h-toolbar` 등)을 사용할 것."**

`index.js` 는 CommonJS 형태로 로드되므로 ESM/TypeScript는 번들링 필수.

---

## 6. Tweak Store

### 6-1. 현재 승인된 트윅 14개

| 트윅 | 작성자 | 기능 |
|---|---|---|
| Bennett's UI Improvements | bennett | 업그레이드 프롬프트 숨김, 사용량/메시지 지표 표시 |
| Better Browser | bennett | 브라우저 사이드 패널 탭/인라인 devtools/내비 단축키 |
| Better Terminal | bennett | 터미널 분할 패널, 네이티브 팝아웃, 메모리 워치독 |
| Codex Tab Switcher | bennett | Ctrl+Tab 오버레이로 최근 채팅 전환 + 미리보기 |
| Context Follow Up | Arconte112 | 답변 하단에 컨텍스트 기반 다음 단계 프롬프트 |
| Custom Keyboard Shortcuts | bennett | 단축키 탐색/재매핑/비활성화 |
| **Disable Escape** | qoli | **CJK IME 조합 중 ESC가 응답을 끊는 문제 차단** |
| Easy Account Switcher | erknvl | 로컬 Codex auth 세션 저장/전환 + 사용량 캐시 |
| File Editor | bennett | 파일 편집기 |
| Goal | bennett | 목표 관리 |
| iOS Simulator | b-nnett | iOS 시뮬레이터 연동 |
| Project Home | bennett | 프로젝트 홈 화면 |
| Reasoning & Exploration Fixes | shivam94 | 추론/탐색 관련 수정 |
| Windows Computer Use | bennett | Windows 컴퓨터 유즈 |

### 6-2. 보안 심사 프로세스 (공급망 공격 방어)

```
1. 사용자가 Settings → Tweak Store → Publish Tweak 에서 GitHub 레포 제출
2. Codex++가 기본 브랜치의 현재 커밋 SHA를 해석
3. 그 SHA를 담은 GitHub 이슈를 관리자 리뷰용으로 생성
4. 관리자가 해당 SHA 시점의 소스를 직접 리뷰
5. manifest에 스토어용 iconUrl이 있는지 확인
6. index.json 에 approvedCommitSha 로 핀 고정하여 등록
7. gh-pages 에 커밋 → GitHub Pages 배포
```

**설치 시 동작:**
- 승인된 커밋 SHA의 GitHub 아카이브 URL에서만 다운로드
- 다운로드한 `manifest.json` 을 검증한 뒤 교체

**업데이트 정책 (의도적으로 수동):**
- 하루 1회 이하로 GitHub Releases 확인 → `state.json` 에 캐시
- 렌더러에는 캐시된 메타데이터만 전달 (`latestVersion`, `releaseUrl`, `updateAvailable`)
- **자동 다운로드/설치/교체 경로가 런타임에 존재하지 않음**

---

## 7. MCP 연동

트윅이 manifest에 MCP 서버를 선언하면, Codex++가 Codex의
`~/.codex/config.toml` 에 관리 블록으로 자동 동기화한다.

### manifest 선언

```json
{
  "id": "com.you.tools",
  "name": "Tools",
  "version": "0.1.0",
  "githubRepo": "you/tools",
  "scope": "main",
  "mcp": {
    "command": "node",
    "args": ["mcp-server.js"],
    "env": { "TOOLS_MODE": "codex" }
  }
}
```

### 생성되는 config.toml

```toml
# BEGIN CODEX++ MANAGED MCP SERVERS
[mcp_servers.tools]
command = "node"
args = ["/absolute/path/to/tweak/mcp-server.js"]
env = { TOOLS_MODE = "codex" }
# END CODEX++ MANAGED MCP SERVERS
```

### 규칙

- **서버 이름**: 트윅 id에서 유도 (`co.bennett.project-home` → `project-home`,
  `com.you.tools` → `com-you-tools`). 충돌 시 `-2`, `-3` 접미사
- **사용자 수동 항목 보호**: 관리 블록 밖에 같은 이름이 이미 있으면 **덮어쓰지 않고 건너뜀**
- **경로 해석**:
  - 절대 `command` → 그대로 사용
  - `./server.js` 같은 상대/파일형 → 트윅 디렉터리 기준 해석
  - `node`, `python`, `uvx` 같은 명령 이름 → 그대로 사용 (`php`도 동일하게 동작)
  - `args` 중 절대경로거나 `-` 로 시작하면 그대로, 나머지는 파일이 존재할 때만 해석
- **활성화 연동**: 활성 트윅만 동기화. 비활성화하면 다음 리로드에서 관리 항목 제거

---

## 8. 네이티브 브릿지 (macOS)

Codex++ 1.0.0의 `api.codex.native` — **메인 프로세스 백엔드이며, 원시 Owl 객체를 트윅에 절대 노출하지 않는다.**

### 지원 기능

- 트윅 소유 `.node` 모듈 로드
- Objective-C++/N-API 심 (Swift, AppKit, Metal, MetalKit)
- 네이티브 자식 패널 (`NSPanel` / `NSWindow` 오버레이 선호)
- Metal 기반 자식 창 오버레이 (`MTKView`)
- 헬퍼 프로세스

### 권한

```json
{ "permissions": ["codex-windows", "native-view", "native-module", "native-helper"] }
```

- `native-view` — `createPanel()`, `attachView()`, 패널/뷰 메서드 호출
- `native-module` — 트윅 소유 `.node` 모듈
- `native-helper` — 헬퍼 프로세스

### 보안

> **네이티브 경로는 반드시 트윅 디렉터리 안에 실제로 존재해야 한다.
> Codex++는 realpath를 검사하므로 심볼릭 링크 탈출은 거부된다.**

### 예시

```js
module.exports = {
  async start(api) {
    const parent = await api.codex.windows.getPrimary();
    this.panel = await api.codex.native.createPanel({
      parentWindowId: parent?.windowId,
      bounds: { x: 80, y: 80, width: 420, height: 260 },
      transparent: true,
    });
  },
  async stop() { await this.panel?.dispose(); },
};
```

> 직접 자식 `NSView` 삽입은 `getCapabilities().native.directViewAttach` 가
> true 일 때만 사용해야 한다.

---

## 9. 설치 및 사용법

### 9-1. 사전 요구사항

- **Node.js 20 이상** (install.sh가 버전 검사함)
- npm, curl, tar
- Codex 데스크톱 앱 설치됨
- macOS / Windows / Linux

### 9-2. 설치 방법 4가지

**① 에이전트 설치 (Codex에게 시키기)**

```text
Inspect and install this for me: https://github.com/b-nnett/codex-plusplus
Tell me where you install it and send me the local path for adding new tweaks.
```

**② Homebrew (macOS)**

```sh
brew install b-nnett/codex-plusplus/codexplusplus
codexplusplus install
```

**③ 스크립트 원클릭**

```sh
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/b-nnett/codex-plusplus/main/install.sh | bash
```

```powershell
# Windows
irm https://raw.githubusercontent.com/b-nnett/codex-plusplus/main/install.ps1 | iex
```

**④ Bun**

```sh
bun install -g github:b-nnett/codex-plusplus
codexplusplus install
```

설치 후 Codex를 다시 실행하고 Settings에서 Codex++ 섹션을 확인한다.

> `install.sh` 는 소스를 `~/.codex-plusplus/source` 에 받아 빌드하고,
> 기존 설치본은 `.previous` 로 백업한다. `sudo` 실행 시 원래 사용자 홈과
> 소유권을 복구한다.

### 9-3. 주요 명령어

| 명령어 | 설명 |
|---|---|
| `codexplusplus install` | Codex 패치 + 런타임 설치 |
| `codexplusplus status` | 설치 버전/패치 상태 |
| `codexplusplus debug` | 앱 경로, 런타임 타입, 경로, 열림 상태, 브릿지 상태 |
| `codexplusplus repair [--force]` | 패치 재적용 |
| `codexplusplus update` | 최신 GitHub 릴리스로 Codex++ 업데이트 |
| `codexplusplus update-codex` | Codex 공식 업데이터 준비 후 재패치 |
| `codexplusplus doctor` | 서명/무결성/권한 진단 |
| `codexplusplus safe-mode [--off]` | 트윅 전체 비활성화 / 해제 |
| `codexplusplus uninstall [--purge]` | 제거 (+ 트윅/설정/로그/백업 삭제) |
| `codexplusplus create-tweak <path>` | 트윅 스캐폴딩 |
| `codexplusplus validate-tweak <path>` | 매니페스트/엔트리 검증 |
| `codexplusplus dev <path>` | 로컬 트윅 개발 링크 |
| `codexplusplus browser --port 8765` | 브라우저 호스트 모드 |

### 9-4. 소스 체크아웃에서 실행

```sh
npm run build
npm test
node packages/installer/dist/cli.js install
node packages/installer/dist/cli.js debug
```

### 9-5. 브라우저 호스트 모드 (실험적)

```sh
codexplusplus browser --port 8765
# → http://127.0.0.1:8765/
```

숨겨진 Codex 창이 비공개 앱 브릿지를 제공하고, React UI는 일반 브라우저 탭에서 열린다.
디버깅과 브라우저 자동화에 유용하다.
단, 이 모드에서는 인앱 브라우저가 iframe 심을 쓰므로 일부 사이트는 임베드를 차단한다.

---

## 10. 플러그인 vs 스킬 vs MCP

**결론: Codex++는 셋 중 하나가 아니라, 그것들을 담는 "호스트"다.**

| 개념 | 해당 여부 | 설명 |
|---|---|---|
| **플러그인** | 가장 가까움 | 정확히는 "플러그인 시스템/로더". 플러그인 하나가 아니라 플러그인을 실행하는 판 |
| **스킬** | 아님 | 스킬은 AI 모델에 주는 지시문 묶음(마크다운). Codex++는 실행 코드 레이어 |
| **MCP** | 부분 지원 | Codex++ 자체는 MCP 서버가 아니지만, 트윅이 선언한 MCP 서버를 config.toml에 동기화 |

정확한 분류:

```
Codex++ = Electron 앱 패처
        + 플러그인 런타임
        + CLI 도구
        + 플러그인 스토어
        + MCP 설정 동기화 도구
        + macOS 네이티브 브릿지
```

유사 프로젝트 대응표:

| 대상 앱 | 대응 프로젝트 |
|---|---|
| Chrome | 확장 프로그램 시스템 |
| VSCode | Extension Host |
| **Discord** | **BetterDiscord** ← 가장 유사 |
| **Spotify** | **Spicetify** ← 가장 유사 |
| iPhone | 탈옥 + Cydia |

---

## 11. API 토큰 필요 여부

**결론: Codex++ 자체는 API 토큰이 필요 없다.**

README 명시: *"It does not replace Codex, proxy your account, or run a separate Codex clone."*

Codex++는 이미 로그인된 Codex 앱 **안에서** 실행되며, AI 호출은 전적으로 Codex가 자체 인증으로 수행한다.

| 상황 | 토큰 필요 | 비고 |
|---|---|---|
| Codex++ 설치/실행 | ❌ | 불필요 |
| Codex로 AI 사용 | ❌ | 기존 Codex 계정 인증 사용 |
| 트윅 업데이트 확인 | ❌ | GitHub 공개 API (비인증 시간당 60회 제한) |
| 트윅 스토어 목록 조회 | ❌ | GitHub Pages 정적 JSON |
| 스토어에 트윅 제출 | 🟡 | GitHub 로그인 (이슈 생성, 브라우저로 진행) |
| 내가 만든 트윅이 외부 API 호출 | ✅ | 해당 트윅이 자체적으로 키 관리 |

> ⚠️ `Easy Account Switcher` 트윅이 "로컬 Codex auth 세션 저장/전환" 기능을 제공하는 데서 알 수 있듯,
> **트윅은 인증 정보에 접근할 수 있다.** 신뢰할 수 있는 트윅만 설치할 것.

---

## 12. GitHub에서 유명한 이유

1. **타이밍** — Codex 데스크톱 앱에 확장 생태계가 전무한 시점의 선점(first mover)
2. **검증된 장르** — BetterDiscord/Spicetify 패턴을 이미 아는 사용자층
3. **해커 감성** — asar 패치, 코드사인 재서명, 무결성 해시 재계산의 기술적 깊이
4. **높은 완성도** — 문서 15개, 단위 테스트, CI, 5개 배포 채널, 크로스플랫폼, 복구 체계(watcher/repair/doctor/safe-mode/backup)
5. **즉각적 체감 가치** — 설치 직후 바로 좋아지는 기능들
6. **AI 시대 마케팅** — 설치법 1번이 "Codex에게 시키기" + `AGENTS.md` 로 AI가 트윅을 대신 작성
7. **커뮤니티 설계** — Discord, 기여자 크레딧, 외부 개발자 트윅 4개 이미 등록, 투명한 심사
8. **플랫폼화** — 단순 도구가 아닌 스토어 기반 생태계(네트워크 효과)

---

## 13. 로컬 에이전트 구축 활용성

**결론: 매우 도움이 된다. 단 "직접"보다는 "간접"으로.**

### 직접 활용 가능한 것

| 항목 | 활용법 | 유용도 |
|---|---|---|
| **MCP 번들링** | manifest에 `mcp` 선언 → config.toml 자동 등록/해제 → 내 에이전트를 원클릭 설치 | ⭐⭐⭐⭐⭐ |
| **UI 레이어 무료 제공** | `api.settings.registerPage()` 한 줄로 설정 화면 확보. Electron 앱 새로 만들 필요 없음 | ⭐⭐⭐⭐⭐ |
| **main scope 권한** | 파일시스템, 자식 프로세스(`native-helper`), 창 제어, CDP 접근 | ⭐⭐⭐⭐ |
| **브라우저 호스트 모드** | Codex UI를 HTTP로 노출 → Playwright/Puppeteer로 Codex 자체를 자동 조종 | ⭐⭐⭐⭐ |

### 간접 활용 (설계 학습)

| 배울 것 | 위치 |
|---|---|
| 플러그인 아키텍처 3층 분리 | `loader` → `runtime` → `sdk` |
| 권한 모델 & 샌드박싱 | `permissions` 13종, `tweak-data/` 격리 |
| 안전한 확장 로딩 | `tweak-lifecycle.ts`, `tweak-discovery.ts` |
| 생명주기 관리 | `start(api)` / `stop()` |
| 핫 리로드 | `chokidar` 파일 감시 |
| 공급망 보안 | 커밋 SHA 핀 고정 |
| 네이티브 통합 | N-API, Objective-C++ 브릿지 |
| 경로 탈출 방지 | `native-paths.ts` realpath 검증 |

### 한계

- Codex 데스크톱 앱 없이는 무용지물
- 독립 실행형 에이전트 프레임워크가 아님
- LLM 오케스트레이션(프롬프트/체인/메모리) 기능 없음
- Codex 업데이트마다 깨질 위험

### 추천 활용 패턴

```
1. Codex++를 "에이전트 UI 셸"로 사용
   → 무거운 로직은 MCP 서버에, UI만 트윅으로
2. 아키텍처만 차용해서 다른 앱에 적용
   → 로더/런타임/SDK 3층 분리는 범용 패턴
3. 에이전트 관제탑 트윅 제작
   → 로컬 에이전트 상태를 Codex Settings에서 모니터링
```

---

## 14. React / PHP 로 만들 수 있나

### React — 가능. 오히려 최적 ✅

Codex UI 자체가 React이며, SDK가 `ReactApi` / `ReactFiberNode` 타입을 제공한다.
preload가 React DevTools 형태의 전역 훅을 설치해서 Fiber 트리 탐색이 가능하다.

```ts
export interface ReactFiberNode {
  type: unknown;
  stateNode: unknown;
  memoizedProps: Record<string, unknown> | null;
  memoizedState: unknown;
  return: ReactFiberNode | null;
  child: ReactFiberNode | null;
  sibling: ReactFiberNode | null;
}
```

**구현 예시:**

```js
// index.js (esbuild로 CJS 번들한 결과)
const React = require("react");
const { createRoot } = require("react-dom/client");
const App = require("./App").default;

module.exports = {
  start(api) {
    api.settings.registerPage({
      id: "main",
      title: "내 React 트윅",
      render(root) {
        this._r = createRoot(root);
        this._r.render(React.createElement(App));
      },
    });
  },
  stop() { this._r?.unmount(); },
};
```

**빌드:**

```json
{
  "scripts": {
    "build": "esbuild src/index.jsx --bundle --format=cjs --outfile=index.js --platform=node"
  }
}
```

**주의 2가지**
1. 반드시 **CommonJS로 번들**(`--format=cjs`). ESM/TS는 변환 필수
2. AGENTS.md 프라임 디렉티브 — Codex Tailwind 토큰(`text-token-*`, `bg-token-*`) 사용, 색상/크기/폰트 하드코딩 금지 (스토어 심사 기준)

### PHP — 부분 가능 🟡

**불가능:** 트윅 본체를 PHP로 작성 (런타임이 `require()`로 JS 모듈만 로드)

**가능한 3가지:**

**① PHP를 MCP 서버로**

```json
{ "mcp": { "command": "php", "args": ["./mcp-server.php"] } }
```

문서상 `node`, `python`, `uvx` 같은 명령 이름은 그대로 실행되므로 `php`도 PATH에 있으면 동작한다.

**② PHP를 백엔드 API로**

```
[PHP 서버 (Laravel/Symfony)] ←── HTTPS ──→ [JS 트윅] ──→ [Codex UI]
 라이선스 인증 / 사용량 집계 / 팀 동기화
```

수익화 시 이 구조가 가장 현실적이다.

**③ PHP 자식 프로세스 스폰**

```js
const { spawn } = require("node:child_process");
module.exports = {
  start(api) {
    this.proc = spawn("php", ["-S", "127.0.0.1:9000", "-t", api.fs.dataDir]);
  },
  stop() { this.proc?.kill(); },
};
```

> `native-helper` 권한 필요. 사용자 PC에 PHP가 설치되어 있어야 함.

### 추천 스택

```
트윅 UI        →  React + TypeScript (esbuild → CJS)
스타일         →  Codex Tailwind 토큰 재사용
로컬 로직      →  Node.js (main scope)
MCP 서버       →  Node / Python / PHP 자유
클라우드 백엔드 →  PHP(Laravel) 또는 Node  ← 라이선스 서버
```

---

## 15. 수익화 아이디어

### 전제: 리스크 체크

| 리스크 | 내용 | 대응 |
|---|---|---|
| 플랫폼 리스크 | OpenAI가 차단하면 즉시 종료 | 종속도 낮은 모델 선택 |
| ToS 리스크 | 앱 변조가 약관 위반 소지 | 법률 자문, 사용자 책임 명시 |
| 시장 크기 | Codex 데스크톱 사용자 수가 상한 | 니치 고가 전략 |
| 무료 경쟁 | 기존 트윅 전부 MIT 무료 | 무료로 불가능한 것을 판매 |
| 유지보수 | Codex 업데이트마다 깨짐 | 구독 모델로 유지비 충당 |

**핵심 전략: Codex++에 종속되지 않는 수익 모델을 고를 것.**

---

### TIER 1 — 최우선 추천

#### ① 기업용 Codex 관리 플랫폼

**해결하는 문제**: 팀 단위로 Codex를 쓸 때 설정 제각각, MCP 서버 파악 불가, 보안 감사 불가, 비용 집계 불가

```
[관리자 웹 대시보드]  ←── HTTPS ──→  [Enterprise Agent 트윅]
 - 설정 일괄 배포                      - 정책 수신 & 강제 적용
 - 트윅 화이트리스트                    - 사용 이벤트 전송
 - MCP 중앙 관리                       - 금지 동작 차단
 - 사용량/비용 집계
 - 감사 로그
```

| 플랜 | 가격 |
|---|---|
| Team (10~50명) | $12/seat/월 |
| Business (50~200명, SSO) | $25/seat/월 |
| Enterprise (온프레미스/SLA) | 연 $30,000~ |

**시뮬레이션:** 고객사 20개 × 40석 × $15 = 월 $12,000 (연 $144,000)
고객사 50개 × 60석 × $18 = 월 $54,000 (연 $648,000)

**장점**: B2B 가격 저항 낮음, 이탈률 낮음, **Codex++ 없이도 독립 제품화 가능**

난이도 ⭐⭐⭐ | 초기 3~4개월 | 추천도 ★★★★★

---

#### ② AI 사용량·비용 분석 SaaS

AI 비용은 오르는데 가시성이 없다. 기존 `Bennett's UI Improvements`의 사용량 표시는 너무 기초적이다.

**기능**: 실시간 토큰 그래프 / 프로젝트·저장소별 비용 배분 / 팀원별 랭킹 /
비효율 프롬프트 자동 탐지 / 모델 다운그레이드 제안 / 월간 PDF 리포트 /
예산 초과 알림(Slack·이메일) / 월말 비용 예측

| 플랜 | 가격 |
|---|---|
| Free | 7일 보관, 1인 |
| Pro | $9/월 |
| Team | $7/seat/월 (최소 5석) |

**시뮬레이션:** Pro 500명 × $9 + 팀 30개 × 12석 × $7 = **월 $7,020 (연 $84,240)**

**확장 포인트**: 데이터가 쌓이면 "업계 평균 대비 효율" 벤치마크를 별도 상품화 가능

난이도 ⭐⭐⭐ | 추천도 ★★★★★

---

#### ③ 프리미엄 트윅 번들

**팔면 안 되는 것** (무료로 이미 존재): 단축키 변경, 테마, UI 정리, 탭 전환

**팔 만한 것:**

| 트윅 | 기능 | 가격 |
|---|---|---|
| Memory Pro | 프로젝트별 장기 기억, 벡터 검색, 세션 간 컨텍스트 연결 | $29 |
| Multi-Agent | 다중 Codex 세션 병렬 실행 + 결과 자동 병합 | $39 |
| Snapshot | 코드 변경 전 자동 스냅샷 + 원클릭 롤백 | $19 |
| Workflow | 노코드 AI 워크플로 빌더 (n8n 스타일) | $49 |
| Voice | 음성 명령 + TTS 응답 | $24 |
| Codebase RAG | 전체 코드베이스 임베딩 + 시맨틱 검색 | $59 |
| Team Sync | 팀 간 프롬프트/컨텍스트 실시간 공유 | $15/월 |

**번들:** Starter $79(3개) / Pro $149(전체+1년) / Lifetime $299

**라이선스 구현:**

```js
// 트윅 main scope — 머신 지문 생성
const machineId = require("node:crypto")
  .createHash("sha256")
  .update(require("node:os").hostname() + require("node:os").userInfo().username)
  .digest("hex").slice(0, 16);

const res = await fetch("https://api.example.com/verify", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ key: licenseKey, machine: machineId }),
});
```

```php
// Laravel 백엔드
Route::post('/verify', function (Request $r) {
    $lic = License::where('key', $r->key)->firstOrFail();
    if ($lic->machines()->count() >= $lic->seat_limit
        && !$lic->machines()->where('id', $r->machine)->exists()) {
        return response()->json(['ok' => false, 'reason' => 'seat_limit'], 403);
    }
    $lic->machines()->firstOrCreate(['id' => $r->machine]);
    return ['ok' => true, 'expires' => $lic->expires_at];
});
```

**결제**: Lemon Squeezy / Paddle (MoR — 세금 처리 자동, 개인 개발자 적합)

**시뮬레이션:** 월 100개 × $35 = 월 $3,500 / 월 300개 × $35 = 월 $10,500

난이도 ⭐⭐ | 추천도 ★★★★

---

### TIER 2 — 보조 수입

#### ④ MCP 서버 구독 번들

| 팩 | 포함 | 타겟 | 가격 |
|---|---|---|---|
| DevOps | AWS, K8s, Terraform, Datadog | 인프라 | $19/월 |
| Frontend | Figma, Storybook, Lighthouse, a11y | 프론트 | $15/월 |
| Data | Snowflake, dbt, Airflow, Jupyter | 데이터 | $19/월 |
| **한국** | 네이버, 카카오, 토스, 쿠팡, 국세청 | 한국 개발자 | ₩19,000/월 |
| Backend | PostgreSQL, Redis, Kafka, gRPC | 백엔드 | $15/월 |

Codex++의 MCP 자동 동기화 덕분에 설치가 원클릭.
**시뮬레이션:** 400명 × $17 = 월 $6,800

난이도 ⭐⭐⭐ | 추천도 ★★★★

---

#### ⑤ 교육 콘텐츠

**Codex++가 사라져도 살아남는 모델.**

```
Module 1. Electron 앱 해부학 (asar, 메인/렌더러, IPC)
Module 2. 앱 패치 기술 (진입점 하이재킹, 무결성 해시, 코드사인)
Module 3. 프리로드 인젝션 (registerPreloadScript, React Fiber, MutationObserver)
Module 4. 플러그인 아키텍처 (3층 분리, 권한 모델, 생명주기, 핫리로드)
Module 5. 네이티브 브릿지 (N-API, Obj-C++, AppKit/Metal)
Module 6. 배포 & 수익화 (라이선스 서버, 스토어 심사)
```

| 상품 | 가격 |
|---|---|
| 인프런/Udemy 강의 | ₩99,000 |
| 자체 코호트 | $299 |
| 1:1 멘토링 (4주) | $999 |
| 전자책 | $39 |

**시뮬레이션:** 인프런 500명 × ₩99,000 × 0.7 ≈ **₩34,650,000**

난이도 ⭐⭐ | 추천도 ★★★★ (리스크 최저)

---

#### ⑥ 커스텀 트윅 제작 대행

| 상품 | 가격 | 기간 |
|---|---|---|
| 간단 트윅 (UI/단축키) | $1,500 | 1주 |
| 표준 트윅 (사내 API 연동) | $5,000 | 3주 |
| 복잡 트윅 (MCP + 네이티브) | $15,000 | 8주 |
| 유지보수 리테이너 | $800/월 | 계속 |

**수요 근거**: 사내 Jira/Confluence/GitLab 연동, 코딩 컨벤션 자동 검사,
디자인 시스템 컴포넌트 생성, 규제 산업(금융/의료) 데이터 유출 차단

**시뮬레이션:** 연 8건 × $6,000 = $48,000 + 리테이너 $9,600

난이도 ⭐ | 추천도 ★★★★ — **가장 빠르게 현금화 가능**

---

### TIER 3 — 장기 / 고위험

#### ⑦ 자체 유료 트윅 마켓플레이스
수수료 20~30%. 결제/라이선스/배포/심사/환불 인프라 필요.
닭과 달걀 문제가 심각(개발자 없으면 유저 없고, 유저 없으면 개발자 없음).
GMV $50k/월 × 25% = 월 $12,500 (단 2~3년 소요)
난이도 ⭐⭐⭐⭐⭐ | 추천도 ★★

#### ⑧ 도메인별 AI 에이전트 팩
"React 리팩토링" $39/월, "Laravel 마이그레이션" $39/월,
"보안 감사" $59/월, "레거시 현대화" $79/월.
MCP + 프롬프트 + 워크플로 패키징. 마진 좋지만 품질 유지 난이도 높음.
난이도 ⭐⭐⭐⭐ | 추천도 ★★★

#### ⑨ 프리미엄 테마/디자인 마켓
개별 $5~15 / 팩 $29 / 커스텀 $199.
경쟁 심하지만 **브랜딩·트래픽 유입 퍼널**로는 우수
(무료 테마로 유저 확보 → 유료 트윅 전환).
난이도 ⭐ | 추천도 ★★

#### ⑩ 한국 시장 특화 ★ 강력 추천
```
한글 IME 완벽 대응        (Disable Escape 트윅이 수요를 이미 입증)
네이버클라우드/카카오/토스 API MCP
국내 SI 프로젝트 표준 코드 생성
한글 주석/문서 자동 생성
국내 보안 규제(전자금융감독규정) 대응
한국어 UI 완전 번역
```
**경쟁자 거의 없음. 로컬라이제이션 = 강력한 해자.**
가격 ₩9,900/월 또는 ₩99,000/년 → 1,000명 = **월 ₩9,900,000**
난이도 ⭐⭐ | 추천도 ★★★★★

---

## 16. 실행 로드맵

```
Phase 1 (0~3개월) — 신뢰 구축 & 현금 확보
  · 무료 트윅 2~3개 공개 → 스토어 등록
  · 기술 블로그 / 유튜브 시작
  · 대행 프로젝트 1~2건 수주 ← 현금 흐름
  목표: 인지도 + $5,000

Phase 2 (3~6개월) — 첫 유료 제품
  · 프리미엄 트윅 1개 출시 ($29)
  · 라이선스 서버 구축 (Laravel + Lemon Squeezy)
  · 한국 특화 기능 추가
  목표: MRR $2,000

Phase 3 (6~12개월) — SaaS 전환
  · 사용량 분석 대시보드 출시
  · 온라인 강의 런칭
  · MCP 번들 구독 시작
  목표: MRR $8,000

Phase 4 (12~24개월) — B2B 스케일업
  · 기업용 관리 플랫폼 출시
  · 첫 엔터프라이즈 고객 확보
  · Codex 외 플랫폼으로 확장 (리스크 분산)
  목표: ARR $300,000+
```

### 개인 개발자 기준 TOP 3

| 순위 | 아이템 | 선정 이유 |
|---|---|---|
| 🥇 | **한국 시장 특화 트윅 팩** | 경쟁자 없음 + 한국어 네이티브 강점 + 낮은 난이도 |
| 🥈 | **커스텀 트윅 제작 대행** | 초기 투자 0원, 즉시 현금화, 레퍼런스 축적 |
| 🥉 | **교육 콘텐츠** | Codex++가 사라져도 지식은 남음 (리스크 제로), 수강생이 곧 고객 |

---

## 17. 리스크 및 주의사항

| 리스크 | 내용 |
|---|---|
| 🔴 비공식 프로젝트 | OpenAI와 무관. ToS 위반 소지 가능성 |
| 🟡 앱 바이너리 수정 | 실제로 앱을 변조. 백업/복구 로직은 견고하나 리스크는 존재 |
| 🟡 임의 코드 실행 | 트윅은 Codex 안에서 임의 코드를 실행. 신뢰 가능한 출처만 설치 |
| 🟡 인증 정보 접근 가능 | 트윅이 로컬 Codex auth 세션에 접근 가능 (Account Switcher 사례) |
| 🟡 업데이트 취약성 | Codex가 asar 레이아웃을 바꾸면 주입 실패 (문서에 명시된 미보호 영역) |
| 🟡 Settings DOM 변경 | 휴리스틱 실패 시 콘솔 경고만 출력, 섹션 미표시 |
| 🟡 안티탬퍼 | 현재 Codex는 런타임 무결성 재검사를 하지 않는 것으로 보이나, 도입되면 대응 필요 |
| 🟡 macOS 편중 | 네이티브 브릿지는 macOS 전용. Windows/Linux는 기본 기능만 |

### 프로젝트가 취한 안전 장치

- 런타임 실패해도 원본 Codex는 항상 실행 (`broken tweak system > broken Codex`)
- 패치 전 원본 백업 (`backup/`)
- SIP / 하드닝 런타임 / 커널 보호 미변경
- `repair` 멱등성 (해시 동일 시 무동작)
- 네이티브 경로 realpath 검증 (심볼릭 링크 탈출 거부)
- 트윅별 파일시스템 샌드박스 (`tweak-data/<id>/`)
- 스토어 커밋 SHA 핀 고정 + 자동 설치 경로 부재
- 사용자 수동 MCP 항목 미덮어씀
- 로그 10MB 상한 (`MAX_LOG_BYTES`)
- macOS 로컬 서명 PKCS#12 비밀번호 자동 생성 및 에러 로그에서 마스킹

---

## 부록 A. 핵심 파일 빠른 참조

| 알고 싶은 것 | 볼 파일 |
|---|---|
| 전체 아키텍처 | `docs/ARCHITECTURE.md` |
| 트윅 작성법 | `docs/WRITING-TWEAKS.md`, `tweaks/AGENTS.md` |
| API 전체 목록 | `docs/tweaks/api-reference.md`, `packages/sdk/src/index.ts` |
| 매니페스트 스펙 | `docs/tweaks/manifest.md` |
| MCP 연동 | `docs/tweaks/mcp.md` |
| 네이티브 브릿지 | `docs/tweaks/native-bridge.md` |
| Owl 런타임 | `docs/OWL-RUNTIME.md`, `docs/OWL-BRIDGE-ROADMAP.md` |
| 문제 해결 | `docs/TROUBLESHOOTING.md` |
| 주입 스텁 실제 코드 | `packages/loader/loader.cjs` (88줄) |
| CLI 명령 구현 | `packages/installer/src/commands/` |
| 스토어 레지스트리 | `store/index.json` |
| 변경 이력 | `CHANGELOG.md`, `docs/releases/` |

## 부록 B. 용어 정리

| 용어 | 설명 |
|---|---|
| **asar** | Electron이 쓰는 아카이브 포맷. JS 파일 수백 개를 하나로 묶음 |
| **Owl** | 현재 macOS Codex 빌드가 쓰는 OpenAI 자체 앱 셸. Chromium + Electron 호환 JS 런타임 |
| **preload** | 렌더러 페이지 로드 직전에 실행되는 스크립트. Node와 DOM에 모두 접근 가능 |
| **Tweak** | Codex++의 플러그인 단위. manifest.json + 엔트리 JS |
| **scope** | 트윅 실행 위치 (`renderer` / `main` / `both`) |
| **MCP** | Model Context Protocol. AI 모델에 외부 도구를 연결하는 표준 |
| **CDP** | Chrome DevTools Protocol. 브라우저를 원격 제어하는 프로토콜 |
| **fuse** | Electron 빌드 타임 기능 플래그. 바이너리 내 플래그로 존재 |
| **Sparkle** | macOS 앱 자동 업데이트 프레임워크. Codex가 사용 |
| **N-API** | Node.js 네이티브 애드온 ABI 안정 인터페이스 |
| **MoR** | Merchant of Record. 판매자 대신 세금/결제를 처리하는 사업자 (Paddle, Lemon Squeezy) |

---

> 이 문서는 저장소 전체(패키지 5개, 문서 15개, 스토어 레지스트리, 설치 스크립트,
> CI 설정)를 실제로 읽고 작성했습니다. 수치와 인용은 저장소 내용 기준입니다. 💖
