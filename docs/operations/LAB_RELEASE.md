# 연구실 영어판 배포 (hyunchul176/hop)

공식 HOP(golbin/hop)에 영어 UI를 더한 비공식 빌드를 이 포크의 Releases로 배포하는 절차다.

## 브랜치

- `english-ui`: 영어 UI 변경만 담는다. 공식 HOP에 PR로 보낼 수 있게 배포 설정을 섞지 않는다.
- `lab-release`: `english-ui` + 이 포크 전용 배포 설정(버전, 업데이트 주소와 공개키, 게시자, 맥 공증 조건). 배포 태그는 이 브랜치에 단다.
- 합칠 때는 merge 를 쓴다. rebase 후 강제 푸시하지 않는다.

## 버전

`공식 major.minor.(공식 patch × 100 + 영어판 번호)`. 예: 공식 0.5.3 기반 첫 영어판은 0.5.300, 같은 기반의 수정판은 0.5.301, 공식 0.5.4 기반은 0.5.400. 숫자만 쓰는 이유는 윈도우 MSI 와 맥 번들이 숫자 버전만 받기 때문이다. 다음 7곳을 같이 바꾼다(`tests/hop-version.test.mjs`가 확인한다).

- `package.json`, `apps/desktop/package.json`, `apps/desktop/src-tauri/tauri.conf.json`
- `apps/desktop/src-tauri/Cargo.toml`, `apps/desktop/src-tauri/Cargo.lock`의 `hop-desktop`
- `apps/desktop/quicklook/Resources/{Preview,Thumbnail}/Info.plist`의 `CFBundleShortVersionString`

## 공식 새 버전 반영

1. `git fetch upstream --tags` 후 `english-ui`에 공식 태그를 merge 하고 충돌을 푼다.
2. `pnpm run test:upstream`, `pnpm run test:studio`, `pnpm run build:studio`. `hop-i18n/coverage.test.ts`가 실패하면 새 화면 문구에 번역 키를 붙인다(`docs/architecture/ENGLISH_UI.md`).
3. `english-ui`를 푸시하고 `gh workflow run ci.yml --ref english-ui`로 Rust 빌드·테스트·clippy 까지 확인한다.
4. `lab-release`에 `english-ui`를 merge 하고 버전을 올린 뒤 커밋·푸시, `vX.Y.Z` 태그를 단다.
5. `gh workflow run hop-desktop.yml --ref lab-release -f macos_arm64=true -f macos_x64=true -f windows_x64=true -f linux_x64=true -f create_release=true -f release_tag=vX.Y.Z -f release_draft=true -f notarize_macos=false`
6. 리눅스 AppImage 점검이 `status 5`(xvfb-run 임시 폴더 정리 실패)로 떨어지면 `gh run rerun <run> --failed`로 다시 돌린다. 앱 문제가 아니다.
7. 초안 릴리스의 `latest.json`이 이 포크 주소와 업데이트 공개키로 서명된 파일을 가리키는지 확인하고, 릴리스 노트를 붙여 공개한다. 공개해야 `releases/latest/download/latest.json`이 새 버전을 낸다.

## 업데이트 서명 키

비밀키는 GitHub 비밀값 `TAURI_SIGNING_PRIVATE_KEY`에 있고, 보관본은 배포 담당자의 `~/.tauri/hop-lab-updater.key`다(암호 없음). 공개키는 `tauri.conf.json`의 `plugins.updater.pubkey`다. 비밀키를 잃으면 이미 설치된 앱은 자동 업데이트를 받을 수 없으니 보관본을 지킨다. 비밀키 내용은 어디에도 출력하거나 붙여 넣지 않는다.
