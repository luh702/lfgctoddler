# 런던순복음교회 유아부 주일 출석체크 (v0.9)

유아부 반별 주일 출석체크 웹앱 — 시범용 프로토타입입니다.

- 단일 HTML 파일(`index.html`), 바닐라 HTML/CSS/JS
- 데이터는 각 기기 브라우저의 `localStorage`에 저장됩니다 (서버 없음)
- 반별 입장(PIN) / 관리자(임원·교역자) 통합 PIN `1004`

배포 주소: https://luh702.github.io/1fgc-nursery-attendance/

## 배포 방식

- GitHub Pages가 `gh-pages` 브랜치의 루트(`/`)를 서빙합니다. `main`과 `gh-pages`는 같은 내용입니다.
- 수정 후에는 두 브랜치를 함께 올립니다.

```sh
git push origin main gh-pages
```
