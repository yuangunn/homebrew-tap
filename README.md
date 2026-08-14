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

### resource 스탠자 갱신

Python 의존성은 formula 안에 `resource` 스탠자로 고정한다. 손으로 쓰지 말고
Homebrew 가 생성하게 한다.

```bash
brew update-python-resources image-trigger-clicker
```

`pyproject.toml` 의 `dependencies` 를 읽어 의존성 트리 전체를 풀고, 각 패키지의
sdist URL 과 SHA256 을 formula 에 써넣는다. 의존성 버전을 올린 뒤에는 반드시 다시 실행한다.

### opencv-python 에서 막히면

`opencv-python` 은 sdist 를 받으면 CMake 로 OpenCV 전체를 컴파일한다. 30분 이상 걸리고
툴체인 문제로 실패하기도 하며, `brew update-python-resources` 자체가 여기서 멈추기도 한다.

그럴 때는 Homebrew 가 이미 빌드해 둔 opencv 를 쓴다.

1. formula 에서 `resource "opencv-python"` 과 `resource "numpy"` 블록을 지운다.
2. `depends_on "numpy"` 와 `depends_on "opencv"` 를 추가한다.
3. `install` 을 바꿔서 brew 의 `cv2` / `numpy` 를 venv 에서 볼 수 있게 한다.

   ```ruby
   def install
     virtualenv_install_with_resources
     site = Language::Python.site_packages("python3.12")
     (libexec/site/"homebrew-deps.pth").write <<~PTH
       #{Formula["opencv"].opt_lib}/#{site}
       #{Formula["numpy"].opt_lib}/#{site}
     PTH
   end
   ```

**트레이드오프**: 설치는 훨씬 빠르고 안정적이지만, brew 의 `opencv` 가 업그레이드되면
이 도구도 함께 영향을 받는다. resource 방식은 버전이 formula 에 고정되어 재현성이 높다.

같은 내용이 formula 파일 상단 주석에도 있다.

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
