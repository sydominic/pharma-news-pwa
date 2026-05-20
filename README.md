# v24-render-npmfix 변경사항

- Render 빌드 실패 원인이 될 수 있는 내부 registry package-lock 파일을 제거했습니다.
- Node 버전을 20.11.1로 고정했습니다.
- npm install이 public npm registry를 사용하도록 `.npmrc`와 install script를 보강했습니다.
- API 버전은 `v24-render-npmfix`입니다.
