# 런던순복음교회 유아부 주일 출석체크 (v1.0)

유아부 반별 주일 출석체크 웹앱 — 시범용 프로토타입입니다.

- 단일 HTML 파일(`index.html`), 바닐라 HTML/CSS/JS
- **v1.0부터 Supabase 실시간 공유 DB를 씁니다.** 한 기기(폰)에서 출석을 찍으면 다른 기기에도 몇 초 안에 그대로 보입니다.
- 반별 입장(잠금을 켜면 4자리 PIN) / 관리자(임원·교역자) 통합 PIN `1004`
- 서버에 연결하지 못하면(지하·비행기 등) 이 기기(`localStorage`)에 먼저 저장하고, 연결되면 자동으로 다시 올립니다.

배포 주소: https://luh702.github.io/lfgc-toddler-attendance/

## 공유 저장 구조 (Supabase)

- 프로젝트: `lfgc-toddler-attendance` (조직 `LFGC`, Free $0 플랜, 리전 `eu-west-2` West Europe — London)
- Project URL: `https://vyvgkmjekenwgjklrsxq.supabase.co`
- 테이블 `public.app_state` — 앱 자료를 key 단위 문서로 나눠 저장합니다.
  | key | 내용 |
  | --- | --- |
  | `meta` | 관리자 계정·관리자 PIN·설정 |
  | `classes` | 반 목록(이름·연령·PIN) |
  | `teachers` | 담당 선생님 명단 |
  | `children` | 원아 명단(생일·첫 방문일·유의사항·사진) |
  | `att:YYYY-MM-DD` | 그 주일(일요일) 출석 기록 |
  | `prayer:YYYY-MM-DD` | 그 주 기도제목과 '함께 기도했어요' |
- 열: `key`(PK) · `data`(jsonb) · `rev`(bigint, 밀리초 시각) · `updated_by`(기기 id) · `updated_at`
- RLS: `anon`·`authenticated` 역할에 select/insert/update/delete 허용 (앱이 회원가입 없이 반별 1탭 입장이라 anon 키로 씁니다)
- 실시간: 같은 채널(`lfgc-attendance-sync`)로 바뀐 key만 방송(broadcast)하고, 받은 기기는 그 key만 다시 읽습니다. `app_state`는 `supabase_realtime` 발행에도 들어 있어 Postgres 변경 알림도 함께 받습니다. 30초마다·화면 복귀 시·온라인 복귀 시 한 번씩 확인합니다.
- 충돌: 같은 주일 출석이 두 기기에서 겹치면 아이별로 더 늦게 찍은 기록(`at`)을 남기고, 기도 '함께 기도했어요'는 합칩니다.
- 접속 정보는 `repos/lfgc-nursery-attendance/.env.local`(git 제외)에 있습니다. **DB 비밀번호와 service_role 키는 절대 이 저장소(index.html 포함)에 올리지 않습니다.**

## 배포 방식

- GitHub Pages가 `gh-pages` 브랜치의 루트(`/`)를 서빙합니다. `main`과 `gh-pages`는 같은 내용입니다.
- 수정 후에는 `sh repos/lfgc-nursery-attendance/deploy.sh`(main·gh-pages 푸시 + HTTP 200 확인)를 씁니다.
