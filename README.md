# yuangunn/homebrew-tap

개인 Homebrew tap.

## 설치

```bash
brew tap yuangunn/tap
brew install image-trigger-clicker
itc doctor
```

## Formula

| 이름 | 설명 |
|---|---|
| [`image-trigger-clicker`](Formula/image-trigger-clicker.rb) | 화면에 지정한 이미지가 나타나면 미리 설정한 좌표를 클릭하는 macOS용 범용 데스크톱 자동화 CLI ([저장소](https://github.com/yuangunn/image-trigger-clicker)) |

### image-trigger-clicker 설치 후

**반드시 `itc doctor` 를 먼저 실행하라.** macOS 권한이 없으면 오류 없이 조용히
아무것도 동작하지 않는다.

- 시스템 설정 > 개인정보 보호 및 보안 > **화면 기록**
- 시스템 설정 > 개인정보 보호 및 보안 > **손쉬운 사용**

권한은 `itc` 실행 파일이 아니라 `itc` 를 실행하는 **터미널 앱**에 부여된다.
터미널 앱을 바꾸면 새 앱에 대해 다시 허용해야 한다.

자세한 사용법은 [메인 저장소 README](https://github.com/yuangunn/image-trigger-clicker#readme) 참고.

---

## 유지보수

### 의존성: 무엇을 휠로, 무엇을 sdist 로

`virtualenv_install_with_resources` 는 모든 resource 를 sdist 에서 빌드한다
(`pip --no-binary=:all:`). 그런데 셋은 그 방식으로 설치되지 않는다.

| 패키지 | sdist 빌드가 안 되는 이유 |
|---|---|
| `numpy` | Apple clang 으로 컴파일이 깨진다. Homebrew 의 numpy formula 도 gcc 로 빌드한다 |
| `opencv-python` | CMake 로 OpenCV 전체를 빌드한다(30~60분). 빌드 의존성이 numpy 라 위 문제를 먼저 만난다 |
| `pillow` | jpeg/tiff/webp 등 이미지 라이브러리 헤더를 요구한다 |

실제로 v0.1.0 을 전부 sdist 로 설치해 보다가 11분 만에 numpy 컴파일 오류로 깨졌다.
그래서 이 셋만 공식 배포 **휠**을 그대로 설치한다(`on_arm` / `on_intel` 블록).
나머지(pyautogui 계열, pyobjc)는 sdist 그대로 두며 1분 안에 빌드된다.

주의: Homebrew 캐시는 파일을 `<sha256>--원래이름` 으로 저장하는데 pip 가 이를
잘못된 휠 파일명으로 거부한다. formula 의 `install` 에서 원래 이름으로 복사한 뒤 넘긴다.

### sdist resource 스탠자 갱신

```bash
brew update-python-resources image-trigger-clicker
```

`pyproject.toml` 의 `dependencies` 를 읽어 의존성 트리를 풀고 sdist URL 과 SHA256 을
formula 에 써넣는다.

**주의**: 이 명령은 `numpy` / `opencv-python` / `pillow` 도 sdist 스탠자로 덮어쓴다.
실행한 뒤에는 그 셋을 휠 형태로 되돌려야 한다.

### 검토했지만 쓰지 않은 대안: `depends_on "opencv"`

Homebrew 가 미리 빌드해 둔 bottle 을 쓰는 방법이다. 빠르고 안정적이지만
brew 의 opencv 5.0 은 의존 formula 를 **106개** 끌고 온다 — Qt, VTK, OpenVINO,
ffmpeg, tesseract, boost, gcc 까지. 수 GB다. (휠 방식의 Cellar 용량은 183MB.)
`cv2.matchTemplate` 하나 때문에 치르기엔 과한 비용이라 쓰지 않았다.

순정 Homebrew 구성을 원한다면 formula 주석과 메인 저장소의
`packaging/homebrew-tap/README.md` 에 전환 방법이 있다.
brew 의 opencv 는 `python@3.14` 용으로 빌드되므로 formula 의 파이썬도 옮겨야 한다.

### formula 검증

```bash
brew audit --strict --online yuangunn/tap/image-trigger-clicker
brew install --build-from-source yuangunn/tap/image-trigger-clicker
brew test image-trigger-clicker
```

### 릴리스 자동 갱신

메인 저장소의 `.github/workflows/update-tap.yml` 이 릴리스 게시 후 이 저장소의
formula `url` / `sha256` 을 자동 커밋한다. 메인 저장소에
`TAP_GITHUB_TOKEN` 시크릿(이 저장소에 `contents: write` 권한이 있는 PAT)이 필요하다.

```bash
gh secret set TAP_GITHUB_TOKEN --repo yuangunn/image-trigger-clicker
```

기본 `GITHUB_TOKEN` 은 다른 저장소에 쓸 수 없으므로 별도 토큰이 반드시 필요하다.
Fine-grained personal access token 으로 이 저장소만 대상으로 발급하는 것을 권장한다.

## 서명·공증

하지 않는다. `itc` 는 `.app` 번들이 아니라 Homebrew 로 설치되는 CLI 실행 파일이라
Gatekeeper 격리 검사 대상이 아니다.
