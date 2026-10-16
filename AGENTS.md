# AGENTS.md — 작업 가이드 (AI 에이전트용)

이 저장소는 **노장현(Janghyun Noh)** 의 개인 사이트(https://janghyunnoh.github.io)와 영문/국문 CV를 관리합니다.
이 문서를 끝까지 읽고, 아래 규칙대로 작업하세요. 사용자는 배포된 사이트만 확인합니다.

---

## 1. 작업 방식 (가장 중요)

- 사용자 요청 → **저장소를 직접 수정 → `master` 에 commit + push**. 사용자에게 파일 편집을 시키지 않는다.
- 답변은 **한국어로 짧게**. 바뀐 문구는 **EN / KO 를 그대로** 보여준다.
- 사이트 문구는 항상 **영어·한국어 둘 다** 수정한다. CV 도 **영문·국문 둘 다** 같은 내용으로 맞춘다.
- CV 를 고치면 **두 PDF 모두 3페이지 이하**여야 한다 (넘치면 문구를 줄여 맞춤).
- 사용자가 "모아서 한 번에 커밋"을 원하면: 작업용 브랜치(예: `cv-wip`)에 쌓고 미리보기 PDF 를 보여준 뒤,
  "커밋/배포" 지시 때 `git merge --squash` 로 `master` 에 **커밋 1개**로 올린다. (`master` 만 배포됨)
- 매 CV 수정 후 가능하면 **빌드한 영문·국문 PDF 를 사용자에게 보여준다.**
- 커밋 메시지 끝에는 세션이 요구하는 attribution 줄을 붙인다.

## 2. 배포 구조

- GitHub Pages, 브랜치 `master`, push 후 1~2분 내 반영. Jekyll(플러그인 없음).
- **CV PDF 는 GitHub Actions 가 자동 빌드**한다 (`.github/workflows/build-cv.yml`).
  - `cv-src/**` 를 push 하면 XeLaTeX(TeX Live 2023)로 `cv_en.tex`, `cv_ko.tex` 를 빌드 →
    3페이지 이하인지 검사 → `assets/cv/Janghyun_Noh_CV.pdf`, `assets/cv/Janghyun_Noh_CV_KO.pdf` 를 봇이 커밋.
  - 이때 `_data/cv_meta.yml` 의 `updated` 날짜(서울 기준)도 자동 갱신 → CV 페이지 "최종 수정일" 표시.
  - **봇이 커밋을 추가하므로, 다음 push 전에 반드시 `git pull` 할 것.**
- 브라우저 캐시 대응: CV PDF 링크에 `?v={{ site.time }}`, 링크 미리보기 이미지에 `?v=N`.

## 3. 사이트 구조

| 위치 | 내용 |
|---|---|
| `_data/profile.yml` | 이름, `status`(이름 아래 "Open to …" 배지), `title`(학위 줄), About Me(`about`/`about_ko`, HTML 링크 포함), 연락처, CV 경로 |
| `_data/news.yml` | Home "Latest News / 최근 소식" (최신순, 3~5개) |
| `_data/publications.yml` | 저널 논문 (`scope: international/domestic`, links: paper/code/docker) |
| `_data/conferences.yml` | 학회 발표 (`venue`/`venue_ko`, `type`, `award`/`award_ko`, `first_author`) |
| `_data/projects.yml` | 프로젝트 카드 9개 (period, title/_ko, org/_ko, summary/_ko, role/_ko, tags, links) |
| `_data/i18n.yml` | UI 라벨 (EN/KO) |
| `_data/activities.yml`, `awards.yml`, `education.yml`, `skills.yml` | 예전 HTML CV 용 백업 (현재 CV 페이지는 PDF 임베드라 직접 표시 안 됨) |
| `_layouts/profile.html` | 공통 레이아웃, 상단 메뉴, 다크모드, og 메타(미리보기 이미지 `assets/img/og-card.png`) |
| `_includes/sections/{home,research,projects,cv}.html` | 페이지별 섹션 |
| `_includes/item-pub.html`, `badges.html`, `link-buttons.html` | 논문/발표 항목, 배지, 링크 버튼 |
| `assets/css/profile.css` | 스타일 (색상 토큰, 다크모드, 3열 카드, `:lang(ko)` 카드 양쪽정렬) |

- 페이지: `/`, `/research/`, `/projects/`, `/cv/` 와 한국어 `/ko/...`. 한국어는 `*_ko` 필드, 없으면 영어로 fallback.
- 링크 버튼은 URL 이 있을 때만 표시. 연락처 순서: **Email → LinkedIn → GitHub → Google Scholar** (사이트·CV 동일).
- 링크 미리보기 이미지는 얼굴 사진 대신 텍스트 카드(`og-card.png`) — 사진으로 되돌리지 말 것.

## 4. CV 구조 (`cv-src/`)

- `cv_en.tex`, `cv_ko.tex` (Awesome-CV), `awesome-cv.cls`, `fontawesome6.sty`. `_config.yml` 에서 빌드 제외.
- 커스텀한 부분:
  - `awesome-cv.cls`: Source Sans 3 를 경로(`/usr/local/share/fonts/custom/`)로 지정, `\linkedin[표시이름]{id}` 지원,
    헤더 연락처 순서 email → homepage → LinkedIn → GitHub → Scholar.
  - 국문: kotex 없이 XeTeX interchar 로 한글(Noto Sans CJK KR) 처리. 이름은 `\name{}{노장현}` (전체 볼드).
  - `\needspace` 로 섹션 고아 방지, 항목 사이 `\vspace{-1mm}`. **불릿 없는 `\cventry{…}{}` 뒤에는 `\vspace{2.5mm}`** (안 그러면 겹침).
  - Projects 는 하나의 tabularx, 연도는 `\makebox[1.9cm][l]{…}` 로 정렬. 같은 해면 `2025` (2025–2025 쓰지 않음).
  - `㈜` 글리프 없음 → `(주)`.
- 로컬 빌드(선택): 폰트 Source Sans 3·Roboto·Font Awesome 6 OTF, `fonts-noto-cjk` 필요.
  Source Sans 3 는 `/usr/local/share/fonts/custom/` 에 둬야 함. `cd cv-src && xelatex cv_en.tex && xelatex cv_ko.tex`
  로컬 빌드가 어려우면 push 후 Actions 결과(PDF)를 pull 해서 확인해도 된다.

## 5. 지금까지 정한 규칙 (사용자 선호)

**포지셔닝**
- 바이오 연구자가 아니라 **AI 를 활용해 현실 문제를 푸는 사람** — AI 직군(AI 연구자, 데이터 사이언티스트) 지원. 국내·해외.
- 문구는 AI/ML/데이터 관점의 **범용 표현**. 바이오 전문용어는 풀어 쓰거나 뺀다 (MSI/MSS, MIMIC 이름, 모델명 나열 등 지양).
- CV 직함: `AI Researcher · Data Scientist` / `AI 연구원 · 데이터 사이언티스트`.
- 사이트 이름 아래: 배지 "Open to AI Researcher & Data Scientist roles" / "AI 연구자 · 데이터 사이언티스트 포지션 구직 중",
  그 아래 "M.S. in Artificial Intelligence, Myongji University" / "명지대학교 인공지능학 석사". (취업 후엔 `status` 비우기)

**정직성**
- 직접 발표 안 한 국제 학회: `Poster Accepted` / `포스터 채택`. 공저자가 발표한 국내 건은 `Presentation` 으로 표기 가능.
- 직접 개발 안 한 도구는 "활용/using" (예: nf-core/sarek pipeline).
- 사용자가 말하지 않은 사실(수치, 세부 역할)은 지어내지 말고 물어볼 것.

**표기**
- 수상명: 한국 학회 "우수" = **Excellent** (Best 는 최우수에만). 예: Excellent Poster Award, Excellent Presentation Award.
- 기관장 명의 상: 상 이름 뒤 괄호. `Research Honor Award (President's Award), Myongji University` / `우수연구상(총장상), 명지대학교`,
  `Merit Award (President's Award), KOTRA` / `표창장(사장상), 대한무역투자진흥공사`. (학력 줄은 "…from the President of Myongji University (at graduation)")
- 협업 기관: 학력에서 처음 풀네임+약어, 경력에선 `(with KDCA)` 식 약어. 국문 경력은 `(서울대병원)`, `(경북대)` 축약.
- 국문 문체: 명사형 종결("~ 개발", "~ 수상"). 프로젝트 설명 끝에 "프로젝트" 붙이지 않음.
- 학력 줄은 `라벨: 내용` 형식 (Thesis / Advisor / GPA / Award / Teaching Assistant / Capstone Project / Leadership).
- 섹션 역할 분리: Education=학위 정보, Experience=연구/인턴, Extracurricular=활동, Honors=수상·장학. **중복 금지.**
- 국외 활동 제목은 활동명(나라는 위치 칸에), 단체명은 설명 줄.
- 프로젝트 제목은 **한국어 공식 과제명 기준** 번역 ("~을 통한" = through). 사이트 카드 제목과 CV 동일.
- 전화번호 넣지 않음. 학부 GPA 넣지 않음(석사 4.44/4.5 만). TOEIC/OPIc 점수 없음. Research Interest 섹션 없음.

**확정된 주요 사실**
- M.S. in AI, 명지대(2024.09–2026.08), 지도 오민식 교수, AIDA Lab. 학위논문: *Annotation-free phenotype prediction using
  knowledge-augmented clustering from single-cell RNA sequencing data* (Briefings in Bioinformatics 1저자, 2026.07).
- B.S. in Data Technology, School of Convergence Software(융합소프트웨어학부 데이터테크놀로지 전공), 2018.03–2024.08.
  Capstone: CatchYou. 학생회 사무국장.
- 경력: Graduate Researcher(2024.09–2026.08), Undergraduate Researcher(2022.12–2024.08), Bigleader AI Intern(청년 AI 인턴십, KOTRA, 2022.07–10).
- 주소 표기: Yangcheon-gu, Seoul, Republic of Korea / 대한민국 서울특별시 양천구.

## 6. 미결정 / 아이디어

- 국외 활동(아이슬란드 워크캠프, 베트남 선교) 함께한 단체명이 있으면 설명 줄에 추가.
- Hugging Face, FastAPI 를 기술에 넣을지 (사용 여부 확인 필요).
- 사이트 코로나19 카드에 방사선 판독문(radiology reports) 반영 여부.
- LinkedIn 공개 URL 을 짧게 바꾸면 CV 표시를 핸들로 변경 가능.

## 7. 검증 팁

- YAML 수정 후: `python3 -c "import yaml; yaml.safe_load(open('_data/projects.yml'))"`
- Jekyll 을 못 돌리는 환경이면 python-liquid 로 include 일부를 렌더하거나, push 후 라이브 사이트로 확인.
- CV: `pdfinfo` 로 페이지 수, `pdftotext -layout` 로 문구, 페이지 이미지로 줄바꿈/겹침 확인.
