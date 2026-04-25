# 📺 YouTube Auto Feed

내가 선택한 채널의 최신 영상만 모아보고, 페이지 안에서 바로 시청할 수 있는 개인 맞춤 유튜브 피드 서비스입니다.

---

## 주요 기능

- **Google 로그인** — 유튜브 프리미엄 계정으로 로그인하여 광고 없이 시청
- **채널 직접 추가** — 원하는 채널을 검색해서 내 피드에 추가
- **클라우드 저장** — Firebase Firestore에 채널 목록 저장, 어떤 기기에서 접속해도 동일한 피드 유지
- **페이지 내 재생** — 영상 클릭 시 유튜브로 이동하지 않고 페이지 안에서 바로 재생 (GPU 가속 최적화 적용)
- **필터링** — 날짜(오늘/7일/30일/전체), 키워드 검색, 최신순/조회수순 정렬
- **더보기** — 하단 더보기 버튼으로 추가 영상 로드 (YouTube API 페이지네이션 연동)

---

## 사전 준비

### 1. Google Cloud Console 설정

1. [Google Cloud Console](https://console.cloud.google.com) 접속
2. 새 프로젝트 생성
3. **YouTube Data API v3** 활성화 (API 및 서비스 → 라이브러리)
4. **Google 인증 플랫폼** 설정
   - OAuth 동의 화면 → 외부 선택
   - 데이터 액세스에서 아래 범위 추가:
     - `youtube.readonly`
     - `userinfo.email`
     - `userinfo.profile`
     - `openid`
   - 테스트 사용자에 본인 Gmail 추가
5. **OAuth 클라이언트 ID** 생성
   - 애플리케이션 유형: 웹 애플리케이션
   - **승인된 JavaScript 출처** (리디렉션 URI 아님!)에 사용할 주소 추가:
     ```
     http://localhost:8080
     https://배포할도메인.com
     ```
   > ⚠️ 승인된 **리디렉션 URI**가 아닌 **JavaScript 출처**에 입력해야 합니다.

### 2. Firebase 설정

1. [Firebase Console](https://console.firebase.google.com) 접속
2. 새 프로젝트 생성
3. 웹 앱 추가 → `firebaseConfig` 값 복사
4. **Firestore Database** 활성화
   - 테스트 모드로 시작
   - 위치: `asia-northeast3 (서울)`

---

## 설치 및 실행

### 파일 설정

`youtube-feed.html` 파일을 열고 아래 두 곳을 수정합니다.

**1. OAuth Client ID 입력** (하단 `<script>` 태그 내)

```javascript
const CLIENT_ID = '여기에_클라이언트_ID_입력';
```

**2. Firebase 설정 입력** (상단 `<script type="module">` 태그 내)

```javascript
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  ...
};
```

### 로컬 실행

Python이 설치된 경우:

```bash
# html 파일이 있는 폴더에서 실행
python -m http.server 8080
```

브라우저에서 `http://localhost:8080` 접속

또는 VS Code의 **Live Server** 플러그인 사용:
- `youtube-feed.html` 우클릭 → **Open with Live Server**

---

## 사용 방법

### 채널 추가

1. Google 로그인 버튼 클릭
2. 사이드바 상단 검색창에 채널명 입력 후 **검색** 클릭 (또는 Enter)
3. 검색 결과에서 원하는 채널의 **+** 클릭
4. 추가된 채널은 자동으로 Firestore에 저장됨 (다른 기기에서도 유지)

### 채널 삭제

- 사이드바 채널 목록에서 채널에 마우스 올리기 → **✕** 버튼 클릭

### 채널 필터

- 사이드바 상단 **채널 필터** 입력창에 채널명 입력 → 실시간 필터링

### 영상 시청

- 영상 카드 클릭 → 페이지 내 팝업 플레이어로 바로 재생
- **유튜브에서 보기 ↗** 버튼 → 유튜브 앱/웹으로 이동
- **ESC** 또는 **✕ 닫기** 또는 바깥 클릭으로 플레이어 닫기

### 필터 및 정렬

| 기능 | 위치 | 설명 |
|------|------|------|
| 날짜 필터 | 피드 상단 탭 | 전체 / 오늘 / 7일 / 30일 |
| 키워드 검색 | 헤더 검색창 | 제목 및 채널명 검색 |
| 정렬 | 헤더 드롭다운 | 최신순 / 조회수순 |
| 더보기 | 피드 하단 버튼 | 추가 영상 로드 (API 페이지네이션) |

---

## 배포 (Netlify 기준)

1. [Netlify](https://netlify.com) 가입
2. `youtube-feed.html` 파일 드래그 앤 드롭으로 배포
3. 배포된 URL (예: `https://my-yt-feed.netlify.app`)을 Google Cloud Console **승인된 JavaScript 출처**에 추가
4. 저장 후 5~10분 대기

---

## 기술 스택

| 항목 | 기술 |
|------|------|
| 프론트엔드 | HTML / CSS / JavaScript (순수) |
| 인증 | Google OAuth 2.0 (GIS) |
| 데이터 | YouTube Data API v3 |
| DB | Firebase Firestore |
| 배포 | Netlify (정적 호스팅) |

---

## 업데이트 내역

### 최신
- 더보기 버튼 추가 (YouTube API 페이지네이션 연동)
- 플레이어 GPU 가속 최적화 (`will-change`, `translateZ(0)`)
- iframe `allow` 속성 보강으로 재생 품질 개선
- 모달 `backdrop-filter` 제거로 렌더링 부하 감소
- 토큰 만료 시 자동 로그아웃 처리

---

## 주의사항

- YouTube Data API는 하루 **10,000 유닛** 무료 할당량이 있습니다. 채널을 너무 많이 추가하거나 자주 새로고침하면 할당량이 소진될 수 있습니다.
- OAuth 앱이 **테스트 모드**인 경우 등록된 테스트 사용자만 로그인 가능합니다. 여러 사람이 사용하려면 Google 심사를 통해 앱을 게시해야 합니다.
- Access Token은 **1시간** 후 만료됩니다. 만료 시 자동으로 로그인 화면으로 돌아갑니다.
- iframe 플레이어는 유튜브 앱 대비 약간의 화질 차이가 있을 수 있습니다. 고화질 시청이 필요한 경우 **"유튜브에서 보기"** 버튼을 이용하세요.