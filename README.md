# 런던순복음교회 유아부 주일 출석체크 (v1.4)

유아부 반별 주일 출석체크 웹앱 — 시범용 프로토타입입니다.

- 단일 HTML 파일(`index.html`), 바닐라 HTML/CSS/JS
- **v1.0부터 Supabase 실시간 공유 DB를 씁니다.** 한 기기(폰)에서 출석을 찍으면 다른 기기에도 몇 초 안에 그대로 보입니다.
- 반별 입장(잠금을 켜면 4자리 PIN) / 관리자(임원·교역자) 통합 PIN `1004`
- 반은 나이대가 아니라 이름과 한 줄 소개로만 구분합니다. 관리 탭 > 반 정보 수정에서 소개를 고치거나 비울 수 있어요.
- 관리자 모드는 아이·선생님을 손잡이(⠿)로 **끌어서 다른 반으로 이동**할 수 있습니다(Pointer Events — PC 마우스·휴대폰 터치 공통). 반 선택(셀렉트)·수정 창은 그대로 보조 수단으로 남아 있어요.
- **v1.3 졸업 예정(v1.3)** — 아이의 생년월일을 넣거나 바꾸면 영국 9월 학기(만 5세) 기준으로 **졸업 예정일과 D-day가 자동 계산**됩니다. 등록·수정 창에서 `자동 계산 / 직접 지정`을 골라 날짜를 직접 지정할 수 있고, `↺ 자동 계산으로 돌아가기`를 누르면 생일 기준으로 돌아갑니다(직접 지정한 값은 `graduation` 필드, 없으면 자동).
- **v1.3 새친구반 원탭 반 배정** — 새친구반 카드 아래 `🐣 반 배정` 줄에서 **믿음·소망·사랑·은혜반 이름을 한 번 누르면** 확인 후 바로 그 반으로 배정되고 Supabase `children` 행에 즉시 저장됩니다. 권한은 기존 규칙 그대로 **관리자(임원)와 새친구반 담당 선생님만**(타 반 읽기 전용 방문자에게는 버튼이 보이지 않음).
- 서버에 연결하지 못하면(지하·비행기 등) 이 기기(`localStorage`)에 먼저 저장하고, 연결되면 자동으로 다시 올립니다.
- **v1.4 📷 등록카드 사진으로 자동 입력** — 새친구 등록·아이 정보 수정 창 맨 위의 `📷 등록카드 사진으로 자동 입력`을 누르면 카메라(또는 앨범)로 종이 등록카드를 찍어 **이름·생년월일·보호자 연락처·첫 방문일·메모(알레르기 등)**를 입력칸에 자동으로 채웁니다. 사진은 브라우저에서 긴 변 1280px JPEG로 줄여 Supabase Edge Function `scan-registration`(Gemini 비전)으로 보내고, 돌아온 값을 채운 뒤 **저장은 담당자가 확인하고 누릅니다.** 키가 없거나 AI가 붐비면 "잠시 뒤 다시" 안내가 뜨고 직접 입력하면 됩니다.
- **v1.4 🎓 졸업 예정자 카드 강조** — 졸업까지 1년 이내(D-365~D-0)인 아이는 카드에 **따뜻한 앰버 테두리·배경**과 `🎓 졸업 예정` 배지, 이미 졸업일이 지난 아이는 `🎓 졸업 대상` 배지로 한눈에 구분됩니다. 1년 넘게 남은 아이는 기존처럼 일반 카드 + `🎓 D-xxx`만 표시합니다.

배포 주소: https://luh702.github.io/lfgctoddler/

## 공유 저장 구조 (Supabase)

- 프로젝트: `lfgc-toddler-attendance` (조직 `LFGC`, Free $0 플랜, 리전 `eu-west-2` West Europe — London)
- Project URL: `https://vyvgkmjekenwgjklrsxq.supabase.co`
- 테이블 `public.app_state` — 앱 자료를 key 단위 문서로 나눠 저장합니다.
  | key | 내용 |
  | --- | --- |
  | `meta` | 관리자 계정·관리자 PIN·설정 |
  | `classes` | 반 목록(이름·소개·PIN) |
  | `teachers` | 담당 선생님 명단 |
  | `children` | 원아 명단(생일·첫 방문일·유의사항·사진) |
  | `att:YYYY-MM-DD` | 그 주일(일요일) 출석 기록 |
  | `prayer:YYYY-MM-DD` | 그 주 기도제목과 '함께 기도했어요' |
- 열: `key`(PK) · `data`(jsonb) · `rev`(bigint, 밀리초 시각) · `updated_by`(기기 id) · `updated_at`
- RLS: `anon`·`authenticated` 역할에 select/insert/update/delete 허용 (앱이 회원가입 없이 반별 1탭 입장이라 anon 키로 씁니다)
- 실시간: 같은 채널(`lfgc-attendance-sync`)로 바뀐 key만 방송(broadcast)하고, 받은 기기는 그 key만 다시 읽습니다. `app_state`는 `supabase_realtime` 발행에도 들어 있어 Postgres 변경 알림도 함께 받습니다. 30초마다·화면 복귀 시·온라인 복귀 시 한 번씩 확인합니다.
- 충돌: 같은 주일 출석이 두 기기에서 겹치면 아이별로 더 늦게 찍은 기록(`at`)을 남기고, 기도 '함께 기도했어요'는 합칩니다.
- 접속 정보는 `repos/lfgc-nursery-attendance/.env.local`(git 제외)에 있습니다. **DB 비밀번호와 service_role 키는 절대 이 저장소(index.html 포함)에 올리지 않습니다.**

## 등록카드 사진 자동 입력 (v1.4)

- Edge Function: `scan-registration` (프로젝트 `vyvgkmjekenwgjklrsxq`, `verify_jwt=false`, CORS `*`)
  - 앱은 `POST {SUPABASE_URL}/functions/v1/scan-registration` 에 `{ imageBase64, mimeType }` 를 보냅니다(헤더 `apikey`·`Authorization: Bearer <publishable key>`).
  - 응답: `{ ok: true, data: { name, birthDate, parentName, parentPhone, firstVisitDate, notes }, model }` · 실패 시 `{ ok: false, error }`.
  - Gemini 키는 **Edge Function Secrets `GEMINI_API_KEY`** 에만 있습니다. `index.html`·README·커밋에는 절대 넣지 않습니다.
  - 모델: 빠른 `gemini-3.1-flash-lite` 로 먼저 읽고, 거의 못 읽었을 때만 `gemini-3.8-flash` 계열로 한 번 더 시도합니다(붐빌 때 대비).
- 배포·테스트 도구는 `supabase/`(git 제외)에 있습니다: `python3 supabase/deploy_edge_function.py`, `python3 supabase/test_scan.py samples/card_photo.jpg`.

## 배포 방식

- GitHub Pages가 `gh-pages` 브랜치의 루트(`/`)를 서빙합니다. `main`과 `gh-pages`는 같은 내용입니다.
- 수정 후에는 `sh repos/lfgc-nursery-attendance/deploy.sh`(main·gh-pages 푸시 + HTTP 200 확인)를 씁니다.
