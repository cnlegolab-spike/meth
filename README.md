# 청라스마트러닝센터 40일 수학 학습실

첨부 계획서의 70강과 복습 5회를 반영한 웹 프로그램입니다. 화면은 GitHub Pages에서 제공하고, 계정과 학습 기록은 Supabase 서버에 저장합니다. 로고 파일은 원본 그대로 사용했습니다.

## 현재 상태

프로그램 소스와 배포 설정이 준비되어 있습니다. 실제 온라인 저장은 아래 Supabase 설정을 마친 후 사용할 수 있습니다. 아직 실제 서비스에 연결하거나 배포하지 않았습니다. 메인 화면에서 학생 페이지와 관리자 페이지를 선택합니다. 체험 모드는 없으며, 서버 설정 전에는 로그인할 수 없습니다.

## 기능

- 센터 발급 학생 아이디·비밀번호 로그인, 본인 비밀번호 변경
- 40일 × 4개 활동 체크와 서버 저장, 날짜·정답 수·시간·학습 메모 기록
- 전체 활동 진도율, 완료 일수, 수강 완료 강의 수, 구간별 진도
- 관리자 학생 전체 진도와 상세 기록 조회, 학생 발급·비밀번호 재설정
- 70강의 개별 인터넷 강의 주소 등록
- 모바일 화면과 키보드 조작

진도율은 완료 활동 / 160 × 100입니다. 하루 완료는 인강·문제·채점·오답 4항목 모두 완료한 경우입니다. 복습일은 8·16·24·32·40일차이며, 40일차는 전체 복습입니다. 수강 완료 강의 수는 일반 학습일의 인강 체크마다 2개씩 계산하고 복습은 제외합니다. 실제 재생 완료 감지는 없으며 학생이 직접 체크합니다.

## 1. Supabase 준비

1. Supabase에서 센터 소유의 프로젝트를 만듭니다.
2. SQL Editor에 `supabase/schema.sql` 전체를 붙여 넣고 한 번 실행합니다.
3. Authentication 설정에서 **신규 사용자 직접 가입을 비활성화**합니다. 학생 생성은 관리자가 서버 함수로 수행합니다.
4. Authentication → Users에서 이메일 `admin@students.center.invalid`로 최초 관리자를 생성하고, 요청하신 초기 비밀번호를 비밀번호 칸에 직접 입력합니다. 이메일 확인 완료 옵션을 선택하세요. 웹사이트에서는 아이디 `admin`으로 로그인합니다. 비밀번호는 GitHub 파일에 입력하지 않습니다.
5. 생성된 사용자의 UUID를 아래 SQL에 입력해 실행합니다. 최초 관리자 권한을 부여하는 작업이며 브라우저에서는 권한을 바꿀 수 없습니다.

```sql
insert into public.profiles(id,name,grade,role)
values ('관리자 사용자 UUID','센터 관리자','','admin');
```

6. `dist/config.js`에 프로젝트 URL과 **publishable 또는 anon 공개 키**를 넣습니다. service_role, secret 키, 관리자 비밀번호는 절대 넣지 마세요. RLS가 학생의 본인 기록만 허용합니다.

```js
window.CENTER_CONFIG = {
  supabaseUrl: 'https://프로젝트주소.supabase.co',
  supabaseKey: '프로젝트의 공개 키'
};
```

## 2. 계정 발급 서버 함수

Supabase CLI를 설치하고 로그인한 후 프로젝트 폴더에서 다음을 실행합니다. PROJECT_REF는 프로젝트 ID로 바꿉니다. GitHub Pages 저장소 하위 경로가 아닌 **출처(origin)** 를 등록합니다. 예: `https://센터계정.github.io`.

```sh
supabase login
supabase link --project-ref PROJECT_REF
supabase secrets set ALLOWED_ORIGINS=https://센터계정.github.io
supabase functions deploy manage-student --no-verify-jwt
```

함수는 플랫폼의 기본 토큰 검사 대신 내부에서 `/auth/v1/user`로 실제 토큰을 검증하고 DB의 관리자 역할을 확인합니다. `SUPABASE_SERVICE_ROLE_KEY`는 Supabase가 함수에 제공하는 서버 환경변수이며 웹사이트에 노출하지 않습니다. 미리보기도 연결할 때는 `ALLOWED_ORIGINS`에 쉼표로 `http://127.0.0.1:4173`을 추가하세요.

학생 아이디는 영문 소문자·숫자·밑줄·하이픈 3~30자입니다. 내부적으로 `아이디@students.center.invalid`를 Auth 식별자로 사용하며 메일을 보내지 않습니다. 비밀번호 분실은 관리자가 재설정합니다. 최초 발급 비밀번호는 10자 이상이며 학생에게 개별 전달하세요.

## 3. GitHub Pages 배포

1. 압축파일을 풀고 최상위의 `index.html`과 `dist`, `supabase`, `.github` 폴더를 함께 GitHub 저장소의 `main` 브랜치에 올립니다. 최상위 `index.html`은 `dist` 안의 실행 파일을 사용하므로 폴더 구조를 유지해 주세요.
2. 저장소 Settings → Pages → Source를 **GitHub Actions**로 선택합니다.
3. Actions의 `Publish math classroom` 완료 후 Pages 주소를 엽니다.
4. 메인 화면의 관리자 페이지에서 `admin`으로 로그인하고 학생 계정을 발급합니다.
5. ‘강의 링크 관리’에 70강의 실제 HTTPS 주소를 입력합니다. 계획서에는 강의 URL이 없어 임의의 링크를 넣지 않았습니다. 외부 강의 사이트의 별도 로그인이나 수강권이 필요할 수 있습니다.

별도 프레임워크 빌드는 필요하지 않습니다. GitHub Actions는 `dist`만 게시하며, DB SQL과 서버 비밀 키는 웹사이트로 게시하지 않습니다. 원본 로고를 포함하므로 저장소 공개 범위는 센터 방침에 따라 정하세요.

## 4. 운영 전 확인

서로 다른 학생 계정 A와 B를 만들고 다음을 확인하세요.

1. A의 체크와 메모 저장 → 로그아웃·다른 기기 로그인 → 동일 기록 확인.
2. B로 로그인 → A의 기록이 보이지 않는지 확인.
3. 관리자 로그인 → A·B 진도와 상세 기록 확인.
4. 학생 권한으로 타 학생 ID의 저장 요청 및 강의 링크 변경 요청이 차단되는지 확인.
5. 관리자 계정 발급, 비밀번호 재설정, 학생 비밀번호 변경 확인.
6. 연결을 끊고 저장하면 성공으로 표시되지 않고 입력이 유지되는지 확인.

온라인 인증·RLS·계정 발급 통합 검증은 실제 Supabase 프로젝트 연결 후 수행해야 합니다. 정적 UI와 계획 데이터 검사는 소스에서 수행했습니다.

Supabase 프로젝트의 정기 백업과 계정 운영은 센터에서 관리하세요. 기본 Auth 세션만 브라우저의 탭 세션 저장소에 보관하며 학습 기록은 DB를 원본으로 사용합니다. 탭을 닫으면 다시 로그인할 수 있습니다.

## 로컬 미리보기

압축파일을 먼저 모두 풀고 **최상위 `index.html`을 더블클릭**하세요. 별도 서버 없이 메인 화면과 학생·관리자 로그인 화면을 확인할 수 있습니다. `dist` 폴더를 지우거나 `index.html`만 다른 곳으로 옮기면 안 됩니다. 학습계획은 함께 제공되는 일반 JavaScript 파일로 불러오므로 로컬 파일로도 실행됩니다.

실제 로그인과 온라인 저장은 Supabase 연결 후 GitHub Pages 주소에서 사용하세요. 계정 생성과 학습 기록은 서버 연결 후에만 사용할 수 있습니다.

기존처럼 Node.js가 있는 컴퓨터에서 `node preview.mjs`를 실행한 후 `http://127.0.0.1:4173`을 열어도 됩니다.

개발 시작 시 복사된 프레임워크 폴더는 배포 경로에서 사용하지 않습니다. 이 프로그램의 실행 소스는 `dist`, DB 및 함수는 `supabase`, GitHub 배포는 `.github/workflows/pages.yml`입니다.

참고: [Supabase 인증](https://supabase.com/docs/guides/auth), [행 수준 접근 제어](https://supabase.com/docs/guides/database/postgres/row-level-security).
