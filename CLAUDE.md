# CLAUDE.md — 강의 노트 웹앱 작업 규칙

이 파일은 Claude Code가 이 저장소를 고칠 때 먼저 읽는 규칙서다.
`index.html` 한 파일이 앱 전체이므로, 아래 규칙을 모르고 손대면 이미 고친 버그가
되살아난다. **특히 "재발 방지 목록"은 모두 실제로 한 번씩 터졌던 버그다.**

---

## 0. 이 앱이 무엇인가

강의를 들으면서 회차별로 노트를 쓰는 웹앱. GitHub Pages로 배포하고 **아이패드 사파리**에서
주로 쓴다. 서버·로그인·계정이 없고 데이터는 전부 브라우저 `localStorage`에 있다.

### 절대 원칙 (바꾸지 말 것)

| 원칙 | 이유 |
|---|---|
| **기록 보존이 1순위** | 서버가 없다. 날아가면 복구 수단이 없다. 기능보다 우선한다. |
| 단일 `index.html` | 빌드 단계 없음. npm·번들러·프레임워크 도입 금지. |
| 외부 의존성 0 (Pretendard CDN 제외) | 오프라인에서도 열려야 한다. |
| 폰트는 Pretendard | 최초 요구사항. |
| 배경은 흰색 (`--bg:#FFFFFF`) | 최초 요구사항. 다크모드 도입 안 함. |
| 아이패드 우선 | 데스크톱 기준으로 키우지 않는다. (§7 참조) |
| 모든 자료는 PDF로 내보낼 수 있어야 함 | 최초 요구사항. 새 화면을 만들면 PDF 경로도 같이 만든다. |
| 회차 이름은 자유 입력 | "1강~30강"도 "welcome~outro"도 쓴다. 번호를 강제하지 않는다. |
| 노트 구조는 일관 | 이름은 자유지만 노트 틀(템플릿)은 고정된 칸 구성을 따른다. |

---

## 1. 파일 구성

```
index.html              앱 전체 (HTML+CSS+JS 한 파일, 약 4,500줄 / 219KB)
version.json            업데이트 알림용 버전 정보 ★ index.html과 반드시 동기화
manifest.webmanifest    홈 화면 추가용 (이름·흰 배경·아이콘)
icon-192.png
icon-512.png
apple-touch-icon.png    폴더 모양 코랄색 아이콘 (애플 "파일" 앱과 구분되도록 선택한 색)
README.md               사용자용 설명서
HISTORY.md              버전 이력 (개발자용)
CLAUDE.md               이 파일
```

### 배포

GitHub Pages에 위 파일을 그대로 올린다. `version.json`은 **앱과 같은 경로**에 있어야 한다
(`fetch("version.json?ts=...")` 상대 경로로 읽는다).

---

## 2. ★ 버전 올리는 절차 (가장 자주 틀리는 부분)

기능을 고치면 **세 곳**을 같은 버전으로 맞춘다. 한 곳만 바꾸면 사용자에게 업데이트 알림이
안 가거나, 없는 업데이트가 있다고 뜬다.

```
① index.html  → const APP_VERSION = "1.20.0";          (섹션 0, 약 886줄)
② index.html  → CHANGELOG 배열 맨 앞에 새 항목 추가     (섹션 0)
③ version.json → version / date / notes
```

### 버전 번호 규칙 (semver 비슷하게, 세 자리 고정)

- **PATCH** (1.19.1 → 1.19.2) — 버그 수정만. 새 기능 없음.
- **MINOR** (1.18.0 → 1.19.0) — 기능 추가·화면 변경.
- **MAJOR** — 아직 쓰지 않았다. 데이터 구조가 호환되지 않게 바뀔 때만.

`cmpVer()`는 세 자리를 숫자로 비교한다. `1.19.10`은 `1.19.9`보다 크다 (문자열 비교가 아니다).

### CHANGELOG 작성 규칙

사용자가 읽는 글이다. 업데이트 직후 팝업에 그대로 뜬다.

- **사용자 말로 쓴다.** "`LESSON_META`에 `templateId` 추가" (✗) → "이전 내용에 알 수 없는 문자가
  뜨던 문제를 고쳤습니다" (○)
- 고친 버그는 **무엇이 어떻게 보였는지**를 쓴다. 원인은 HISTORY.md에 쓴다.
- 기록이 보존된다는 점을 밝힌다. 데이터 관련 수정이면 "실제로 쓰신 내용은 그대로 보존됩니다"처럼.
- `date`는 `YYYY-MM-DD`.

```js
const CHANGELOG = [
  { v:"1.20.0", date:"2026-11-07", notes:[
    "PDF 상세 설정이 생겼습니다 — 설정 → PDF 모양 → 상세 설정, 또는 강의·노트 화면의 'PDF 설정'",
    "글자 크기(8~16pt), 제목 크기, 줄 간격을 바꿀 수 있습니다",
    ...
  ]},
  ...   // 새 항목은 맨 앞에
];
```

### 업데이트 알림이 작동하는 방식

```
① 켤 때   bootUpdateNotice()
   settings.lastSeenVersion < APP_VERSION  →  변경 내역 팝업
   그 다음 lastSeenVersion = APP_VERSION 으로 올리고 즉시 commit()

② 켤 때 + 30분마다   checkUpdate(false)
   version.json 의 version > APP_VERSION  →  "새 버전이 있습니다" 팝업 + 새로고침 버튼
```

즉 ①은 *이미 받은* 새 버전을 알리고, ②는 *아직 안 받은* 새 버전을 알린다. 둘 다 필요하다.
`version.json`만 올리고 `index.html`을 안 올리면 사용자는 영원히 "새 버전이 있습니다"를 본다.

---

## 3. 코드 지도

`index.html`은 주석으로 구분된 섹션 순서를 지킨다. 새 코드는 **해당 섹션 안에** 넣는다.

### CSS (`<style>`)

| 줄 | 섹션 |
|---|---|
| 22 | 토큰 — CSS 변수, 전역 리셋 |
| 63 | 레이아웃 — 래퍼 폭, 상단바 |
| 128 | 폴더 그리드 — 홈 화면 |
| 151 | 리스트 — 최근 노트 / 회차 목록 |
| 220 | 노트 편집 |
| 263 | 탭바 |
| 283 | 시트 / 모달 / 토스트 (643 부근에 PDF 상세 설정 화면 `.po-*`) |
| 725 | 넓은 화면(아이패드) — **늘리지 않고 촘촘하게** |
| 756 | 인쇄 (PDF 내보내기) — `@media print` + 인쇄 문서 규칙 `.pdoc …` |

### JS (`<script>`)

| 줄 | 섹션 | 내용 |
|---|---|---|
| 883 | **0. 버전** | `APP_VERSION`, `CHANGELOG` |
| 1069 | **1. 저장소** | `KEY`, `BAK`, `Store.read/write`, `uid`, `blank`, `S` |
| 1139 | **2. 자동저장** | `touch/commit/flush`, 실패 배너, 재시도, 다른 탭 감지 |
| 1241 | **3. 유틸** | `esc`, 날짜 포맷, 색 계산(HSL), 구분점, 이모지, **PDF 상세 설정 값(`PDF_OPT_DEFAULT`, `pdfOpt`, `pdfPage`, `pdfVars`)**, 마크다운 편집, 템플릿/회차 헬퍼, `migrate` |
| 1621 | **4. 아이콘** | `I.{back,more,chev,…}` 인라인 SVG 문자열 |
| 1644 | **5. 라우터** | `route()`, `go()`, `hashchange` |
| 1671 | **6. 렌더** | `renderHome/All/Course/Note/Template/PdfDoc/PdfOpt/Emoji/Bullet/Settings`, `fitPoPreview` |
| 3238 | **7. 레이어** | `sheet()`, `toast()`, `toastUndo()`, `confirmSheet()`, `fitViewport()` |
| 3451 | **8. PDF** | `mdInline/mdToHtml/mdBlock`, `pField`, `pdfItems/coursePieces/courseBody`, `applyPrintOpt`, `doPrint`, `pdfNote/Course/All` |
| 3735 | **9. 백업/가져오기** | `exportJSON`, `applyImport` |
| 3825 | **10. 업데이트 확인** | `cmpVer`, `checkUpdate`, `changelogSheet`, `bootUpdateNotice` |
| 3883 | **11. 이벤트** | `click` 위임 핸들러 하나 (`switch(data-act)`) |
| 4469 | **12. 시작** | `applyBullets → render → bootUpdateNotice → checkUpdate` |

줄 번호는 참고용이다. 고치면 밀린다. 찾을 때는 섹션 주석(`/* ===... N. 이름 */`)으로 찾는다.

### 화면 전환 (해시 라우팅)

```
#/                        홈 (강의 폴더 그리드)
#/all                     전체 노트
#/settings                설정
#/c/{courseId}            강의 안 (회차 목록)
#/c/{courseId}/l/{lessonId}  노트 작성
#/t/{templateId}          노트 템플릿 편집
#/pdfdoc                  PDF 문서 서식 편집
#/pdfopt                  PDF 상세 설정 (미리보기·내보내기 = 전체)
#/pdfopt/c/{courseId}     PDF 상세 설정 — 그 강의
#/pdfopt/c/{courseId}/l/{lessonId}  PDF 상세 설정 — 그 노트
#/emoji                   자주 쓰는 이모지
#/bullet                  목록 구분점 (들여쓰기 단계별 기호)
```

`hashchange`에서 `dirty`면 먼저 `commit()`한다. 화면을 옮기다 내용을 잃지 않기 위한 장치다.

### 이벤트는 위임 하나로 처리한다

`document`에 `click` 한 번만 걸고 `data-act` 값으로 분기한다. 버튼에 `onclick`이나 개별
`addEventListener`를 붙이지 않는다. `render()`가 `innerHTML`을 갈아끼우므로 개별 리스너는 사라진다.

```html
<button data-act="del-course" data-id="${c.id}">삭제</button>
```

```js
case "del-course": ... break;   // 섹션 11 switch 에 추가
```

새 `data-act`를 추가하면 `audit_all.js`가 자동으로 그 버튼을 눌러본다 (§9).

---

## 4. 데이터 모델

### 저장 위치

```js
const KEY = "lecture-notes.v1";       // 본 저장본
const BAK = "lecture-notes.v1.bak";   // 직전 세대 (한 세대 전으로 되살리기 위함)
```

`Store.write()`는 **새로 쓰기 전에 기존 값을 `BAK`로 옮긴다.** 저장본이 깨지면
`Store.read()`가 `BAK`에서 되살리고 `restored` 플래그를 세운다. 용량이 모자라면
`BAK`를 비워서라도 본 데이터를 지킨다. 이 순서를 바꾸지 말 것.

### 구조 (schema 2)

```js
{
  schema: 2,
  templates: [{
    id, name,
    cols: 1|2|3,                       // 세로 단(열) 개수
    fields: [{ k, label, hint, ph,
               col: 0..3,              // 0 = 가로 전체, 1..cols = 해당 단
               rows: 1..30 }],         // 높이 (줄 단위)
    createdAt, updatedAt
  }],
  courses: [{
    id, title, source,
    color: "#rrggbb",                  // 자유 지정. 뒷면·그림자는 HSL로 자동 계산
    templateId,
    lessons: [
      { id, type:"section", label, createdAt, updatedAt },
      { id, type:"lesson", label,
        status:"todo"|"doing"|"done",
        templateId?,                   // 이 회차만 다른 템플릿을 쓸 때
        [fieldKey]: "본문",            // ★ 사용자 내용은 필드 키로 직접 들어간다
        createdAt, updatedAt }
    ],
    createdAt, updatedAt
  }],
  settings: {
    lastSeenVersion, savedAt,
    defaultTemplateId,
    pdfStyle: "template"|"text",
    pdfDoc: { cover, section, note },  // PDF 문서 서식 (마크다운 템플릿)
    pdfOpt: { fontSize, headScale, lineHeight,   // PDF 상세 설정 — 읽을 땐 반드시 pdfOpt()
              paper, orient, margin, cover, noteBreak, skipEmpty },
    emojis: [{ id, e, n }],            // 최대 40개
    bullets: [6]                       // 들여쓰기 단계별 기호
  }
}
```

### ★ 내부 키 규칙 (반드시 지킬 것)

회차 객체에는 **앱이 쓰는 키와 사용자 내용이 같은 평면에 섞여 있다.** 그래서 어느 키가
내용인지 판별하는 장치가 필요하다.

```js
const LESSON_META = { id:1, type:1, templateId:1, label:1, status:1, createdAt:1, updatedAt:1 };
const isInternalKey = k => !!LESSON_META[k] || /(Id|At)$/.test(k);
```

> **회차 객체에 새 내부 키를 추가하면 `LESSON_META`에도 반드시 넣는다.**
> 빠뜨리면 그 값이 "사용자가 쓴 내용"으로 취급돼 ① 노트에 내부 ID가 그대로 보이고
> ② 작성 개수가 부풀고 ③ 검색에 걸리고 ④ PDF에 찍혀 나간다.
> 실제로 `templateId`를 빠뜨려 `mvjbnn0stm893`이 노트 내용으로 표시된 적이 있다 (v1.16.1).
> `/(Id|At)$/` 정규식은 그래도 새어 나올 경우를 막는 2차 방어선이다. 1차 방어선에 의존하지 말 것.

필드 키는 `f_` + `uid()` 꼴이다. **한 번 만든 필드 키는 절대 바꾸지 않는다.** 키가 곧 내용의
주소다. 템플릿을 복제·수정해도 기존 키를 유지해야 이미 쓴 노트가 보인다 (v1.4.0에서 터진 버그).

### `migrate()` — 방어적 정규화

앱 시작과 가져오기 직후에 돈다. 역할은 두 가지다.

1. **구버전 데이터 변환** — 예전 `w`(너비) 설정을 `col`(단 배치)로, `size`를 `rows`로,
   색 이름을 hex로. 변환은 **한 번만** 한다 (`if(t.cols === undefined)` 가드).
2. **손상 데이터 보정** — 빠진 `id`·`title`·배열을 채운다. 깨진 백업을 가져와도 앱이 열려야 한다.

끝에서 `Store.write(S)`로 즉시 보존하되 **`savedAt`은 건드리지 않는다.** 마이그레이션은
사용자의 편집이 아니므로 "저장 시각"을 갱신하면 거짓 정보가 된다.

새 설정 키를 추가하면 `migrate()`에 기본값 채우는 줄을 넣는다. 안 넣으면 구버전 데이터에서
`undefined`로 읽혀 터진다.

---

## 5. ★ 기록 보존 규칙 (1순위)

### 자동저장 흐름

```
입력 → touch()         dirty = true, 600ms 디바운스
     → commit()        savedAt 기록 → Store.write() → 성공/실패 분기
```

```js
function commit(){
  const stamp = nowISO(); const prev = S.settings.savedAt;
  S.settings.savedAt = stamp;
  const ok = Store.write(S);
  if(ok){ dirty = false; lastGoodAt = stamp; if(saveFailed){ saveFailed=false; renderSaveBanner(); } }
  else {
    S.settings.savedAt = prev;        // ★ 실패하면 시각을 되돌린다
    dirty = true;
    if(!saveFailed){ saveFailed=true; renderSaveBanner(); }
    scheduleRetry();                   // 5초마다 다시 시도
  }
  paintSaveState();
  return ok;
}
```

> **저장 실패를 성공으로 보이게 두면 안 된다.** v1.16.0 전에는 `Store.write()`의 반환값을
> 무시해서 아무것도 안 써졌는데 "저장됨"이 떠 있었다. 사용자가 믿고 닫으면 전부 날아간다.
> 실패하면 ① `savedAt` 롤백 ② 화면 상단 경고 배너 ③ 백업 내보내기 버튼 ④ 5초마다 재시도.

### 화면을 떠나는 모든 경로에서 저장한다

```js
document.addEventListener("visibilitychange", ...);  // ★ document 에 걸어야 한다
addEventListener("pagehide",     flush);
addEventListener("beforeunload", flush);
addEventListener("blur",         flush);
addEventListener("freeze",       flush);
```

> `visibilitychange`는 **`document`에서 발생한다.** `window`에 걸면 한 번도 불리지 않는다
> (v1.16.0에서 고친 버그 — 앱 전환 시 저장이 안 됐다). iOS 사파리는 이 이벤트를 놓치는
> 경우도 있어 `blur`·`pagehide`를 함께 건다. 하나라도 빼지 말 것.

### 다른 탭을 말없이 덮어쓰지 않는다

`storage` 이벤트로 다른 탭의 저장을 감지한다. 여기서 쓰던 내용이 있으면(`dirty`) 어느 쪽을
남길지 사용자에게 묻고, 없으면 조용히 최신으로 맞춘다.

### 삭제는 되돌릴 수 있어야 한다

강의·회차·섹션·템플릿 삭제, 전체 초기화는 `toastUndo(msg, snapshot)`으로 **8초간 되돌리기**를
띄운다. 새 삭제 기능을 만들면 같은 패턴을 쓴다.

```js
const snap = JSON.parse(JSON.stringify(S));   // 삭제 전 스냅샷
... 삭제 ...
toastUndo("회차를 지웠습니다", snap);
```

### 내용을 글자 수로 자르지 않는다

사용자가 쓴 글에 `.slice(0, n)`을 쓰지 않는다. 표시가 넘치면 **CSS로** 처리한다
(`text-overflow:ellipsis`). v1.19.2에서 뒤로가기 버튼의 강의 제목이 `slice(0,10)` 때문에
"생각을 현실로 만드"로 잘려 보였다. 요약·미리보기 목적이라도 저장된 값은 손대지 않는다.

---

## 6. ★ 재발 방지 목록

모두 실제로 터졌던 버그다. 관련된 코드를 고칠 때 먼저 읽는다.

### iOS / 아이패드

| 규칙 | 안 지키면 |
|---|---|
| **입력 중에 포커스된 칸의 높이를 바꾸지 않는다** | iOS가 한글 조합·자동수정·**텍스트 대치**를 취소한다. `grow()`는 입력 중엔 늘리기만 하고, 전체 재계산은 `blur`에서 한다. (v1.12.0) |
| **`autocorrect="on" autocapitalize="sentences"`를 입력칸에 넣는다. `spellcheck="false"` 금지** | 아이폰 키보드 단축어(텍스트 대치)가 안 먹는다. |
| **IME 조합 중 Enter를 가로채지 않는다** | 조합 중인 음절이 다음 줄로 딸려간다. "`- 테스트`" + Enter → "`- 트아`". iOS는 `isComposing`이 `false`로 오는 일이 있어 **`keyCode === 229`와 `compositionstart/end`로 따라간 `imeOn`을 함께** 본다. 조합 중이면 `return false`하고 브라우저가 줄바꿈을 넣은 뒤 `markerAfterEnter()`를 부른다. (v1.18.0) |
| **`textarea.value`에 직접 대입하지 않는다** | 되돌리기(Cmd+Z) 이력이 날아가고 IME가 끊긴다. `editTA()`를 쓴다 — `execCommand("insertText")` → `setRangeText` → 직접 대입 순으로 내려간다. 사파리에서 되돌리기가 되는 건 첫 번째 경로뿐이다. (v1.17.0) |
| **높이에 `100vh`를 쓰지 않는다** | iOS는 주소창·키보드를 포함해 계산해서 시트 아래가 잘린다. `visualViewport`를 재서 `--vvh`/`--vvt`에 넣고 그 값을 쓴다. 긴 시트의 마지막 `.btn-row`는 `position:sticky`로 고정한다. (v1.12.0) |
| **툴바 버튼에 `type="button" tabindex="-1"` + `mousedown` preventDefault** | 버튼이 포커스를 가져가 서식 막대가 닫히고 키보드가 내려간다. `touchend` 경로도 함께 둔다. (v1.14.0) |
| **입력칸 `font-size`는 16px 이상** | iOS가 자동 확대한다. |

### 렌더 / CSS

| 규칙 | 안 지키면 |
|---|---|
| **인라인 SVG에는 크기 CSS를 반드시 준다** | 설정 화면 아이콘이 300×150px로 떴다. 전역에 `svg{width:20px;height:20px;flex:none}` 안전망이 있지만 개별 지정이 원칙이다. (v1.0.1) |
| **중첩 목록은 `ul ul`에 `padding-left`를 준다** | 커스텀 구분점을 쓰려고 `.md-view ul{padding-left:0}`으로 두면 **미리보기 들여쓰기가 전부 평평해진다.** `.md-view ul ul{padding-left:1.45em}` 및 인쇄용 등가물이 필요하다. (v1.19.0) |
| **넓은 화면에서 키우지 말고 촘촘하게** | 아이패드에서 폴더·설정이 전부 커져서 이상해진다. 섹션 691의 `@media(min-width:700px)`는 **밀도를 올리는** 쿼리다. 폴더는 `auto-fill` + 고정 타일(약 108px)로 개수를 늘린다. |
| **단(열)을 1단으로 줄일 때 `col`을 전부 덮어쓰지 않는다** | 2단 → 1단 → 2단으로 돌아오면 배치가 날아간다. `if(n > 1)` 가드. (v1.4.0) |
| 전체 폭은 1200px, 노트 본문 1040px, 설정 720px | 아이패드 가로에 맞춘 값이다. (v1.4.1) |
| **인쇄 CSS의 글자 크기는 `calc(var(--pfs) * N / 10.5)`로 쓴다** (제목류는 `* var(--phs)`까지) | 고정 pt로 쓰면 PDF 상세 설정의 글자 크기가 그 요소에만 안 먹는다. N은 "본문 10.5pt일 때의 pt". (v1.20.0) |
| **`.p-note`처럼 여러 겹을 감싸는 요소에 태그 선택자를 걸 때는 `>`로 좁힌다** | `.p-note h3`(칸 이름용)이 회차 제목·노트 속 소제목까지 8pt 회색 대문자로 만들었다. 지금은 `.p-note > h3`. (v1.20.0) |
| **본문 규칙은 `.p-field .val`과 `.p-note > .val` 둘 다에 건다** | '내용만' 모양은 칸 상자 없이 `.p-note > .val`에 넣는다. 한쪽만 걸면 그 모양에서 제목이 화면용 27px, 목록이 기본 글머리표로 찍힌다. (v1.20.0) |
| **인쇄 DOM 클래스 이름이 화면 규칙과 겹치지 않는지 본다** | 빈 칸의 `.empty`가 홈 화면의 `.empty{text-align:center}`를 받아 칸 이름이 가운데로 갔다. `.pdoc .p-field{text-align:start}`로 막았다. (v1.20.0) |
| **좁은 화면용 1열 그리드도 `grid-template-columns:minmax(0,1fr)`** | 열 지정이 없으면 열이 내용(줄이기 전 용지 폭 822px)만큼 늘어나 가로 스크롤이 생긴다. (v1.20.0) |

### 마크다운 / 편집

| 규칙 | 안 지키면 |
|---|---|
| **들여쓰기 깊이에 상한을 둔다** | 글자는 10단계 들어가는데 화면은 5단계까지만 보여줘서 글과 화면이 어긋난다. `maxIndent = (BULLET_DEPTHS - 1) * INDENT.length`. (v1.19.1) |
| **빈 줄에도 서식이 걸려야 한다** | 먼저 `#`·`-`를 눌러 두고 쓰는 사용법이 막힌다. `mdLine`에 단일 빈 줄(`only`) 케이스가 있다. (v1.14.0) |
| **번호 재매김 후 커서를 길이 차이로 옮기지 않는다** | 커서가 번호 사이로 끼어든다. (줄 인덱스, 줄 안 오프셋)을 기억하고 기호 길이 변화만큼 다시 매핑한다. (v1.17.0) |
| **노트에 저장되는 글자는 항상 `-`** | 사용자가 정한 구분점(→, ⇒ …)은 **표시와 PDF에만** CSS `--bu0..5`로 적용한다. 저장된 텍스트를 바꾸면 설정을 되돌릴 때 글이 깨진다. (v1.17.0) |
| `{br:true}` 는 단일 줄바꿈 보존 | 빈 줄 없이 줄만 바꿔도 붙지 않아야 한다. 노트 본문 렌더는 전부 이 옵션을 쓴다. |
| `{fold:key}` 는 제목 접기 | `<details>` 기반. 접힌 상태는 화면을 옮겨도 유지된다. **PDF는 접힘과 무관하게 항상 전부 나간다.** |
| 코드 표기(`` ` ``) 안에서는 다른 서식이 걸리지 않는다 | |

### 용어 (두 번 오해해서 재작업한 것)

- **"토글"** = 접었다 펼치기. 제목 수준(H1/H2/H3) 바꾸기가 아니다. 대제목을 접으면 그 아래
  중제목·소제목·본문이 함께 접히는 **위계 구조**를 말한다.
- **"단"** = 세로 열(column). 가로 칸 너비가 아니다.
- 요구사항이 모호하면 **만들기 전에 묻는다.** 이 두 건은 먼저 묻지 않아서 전부 다시 만들었다.

---

## 7. 디자인 규칙

```css
:root{
  --bg:#FFFFFF;        --surface:#F5F6F8;   --surface-2:#EFF1F4;  --line:#E9EBEF;
  --ink:#0E1116;       --ink-2:#4A5058;     --muted:#8C929B;
  --accent:#2F6BFF;    --accent-soft:#EEF3FF;
  --danger:#E5484D;    --ok:#12A594;
  --r-lg:20px;  --r-md:14px;  --r-sm:10px;
  --pad:20px;   --tab-h:64px;
}
```

새 색을 하드코딩하지 않고 토큰을 쓴다. 폴더 색만 예외 — 사용자가 자유 지정하고,
뒷면·중간면·그림자는 **HSL로 변환해서 계산**한다. 그래서 어떤 색을 골라도 명암 관계가
같게 유지된다 (hex를 직접 섞으면 탁해진다, v1.6.0).

기본 폴더 색 목록:
```js
const COLOR_PRESETS = ["#ff8a4d","#50b1f7","#4c86f5","#7a5af0","#14a896","#e75b60"];
```
맨 앞 코랄은 앱 아이콘과 같은 색이다 (애플 "파일" 앱 아이콘과 구분하려고 고른 색).

---

## 8. PDF 내보내기

`window.print()` + `@media print`로 만든다. 별도 라이브러리 없음.

- `#printRoot`(`class="pdoc"`)에 인쇄용 DOM을 만들고 화면 UI는 전부 `display:none`.
- **인쇄 문서 규칙은 `@media print` 밖, `.pdoc …` 아래에 있다.** 실제 인쇄(`#printRoot`)와 PDF 상세
  설정의 쪽 미리보기(`.po-page .pdoc`)가 같은 규칙을 쓰기 위해서다. 인쇄 규칙을 새로 쓸 때도
  `.pdoc`으로 시작한다. `@media print` 안에는 화면 UI 숨기기와 `@page`만 둔다.
- 글자 변수: `--pfs`(본문 크기, pt) · `--plh`(줄 간격) · `--phs`(제목 배율). `doPrint()` 직전에
  `applyPrintOpt()`가 `#printRoot`에 넣는다. 기본값(10.5pt · 1.62 · 1)은 v1.19까지의 고정값과 같다.
- 용지·여백: `@page`에는 변수를 못 쓰므로 `applyPrintOpt()`가 `<style id="pageStyle">`에
  `@page{size:Wmm Hmm;margin:…}`를 다시 쓴다. CSS의 `@page{size:A4;margin:16mm 15mm}`는 기본값일 뿐이다.
  용지는 이름(B5 등) 말고 mm로 준다. 사파리는 `@page size`보다 인쇄 창의 용지를 따를 수 있다.
- PDF 상세 설정은 **읽을 때 반드시 `pdfOpt()`** 로 읽는다 (범위 검사·기본값 채움). 쪽 넘김·빈 회차
  빼기는 `coursePieces()`/`pdfItems()` 한곳에서 정하고, 인쇄와 미리보기가 같이 쓴다.
  노트 하나 PDF(`pdfNote`)는 글자·용지만 쓰고 표지·쪽 넘김·빈 회차 설정은 쓰지 않는다.
- 하이라이트(`<mark>`)는 `print-color-adjust:exact`가 있어야 노란 배경이 인쇄된다.
- 긴 칸은 `.p-field.long`으로 `break-inside:auto` — **페이지를 넘어 이어져야 하고 잘려선 안 된다.**
- 제목류는 `break-after:avoid` (제목만 남고 내용이 다음 장으로 가는 것 방지).
- `pdfStyle:"template"`은 칸 테두리·라벨·단 배치를 화면과 같게 유지한다 (빈 칸도 점선 상자로
  남겨 인쇄해 손으로 채울 수 있게). `"text"`는 내용만.
- 단위는 `mm`를 쓴다 (인쇄 기준).

새 화면·새 필드를 만들면 **PDF 경로에도 반영한다.** 섹션 8의 `pField`/`noteFieldsHTML`/
`coursePieces`를 함께 고친다.

---

## 9. 검증 방법

> **측정하지 않고 "고쳤다"고 하지 않는다.** 이 프로젝트에서 여러 번, 고쳤다고 생각한 것이
> 안 고쳐져 있었다. 사용자가 같은 버그를 두 번 보고한 적이 두 번 있다.

### 테스트 방식

> **테스트 스크립트는 이 저장소에 들어 있지 않다.** 개발 당시 일회성 검증용으로 별도 환경에서
> 돌렸다. 아래는 그 방식과 검사 범위이고, 필요하면 같은 패턴으로 다시 만든다.

`jsdom`에 `index.html`을 올리고, `<script>`를 실행한 뒤 전역 함수와 DOM을 직접 두드린다.
`localStorage`는 가짜 객체로 바꿔 끼워 저장 실패까지 재현한다.

```js
const { JSDOM } = require("jsdom");
const html = require("fs").readFileSync("index.html", "utf8");
const dom = new JSDOM(html, { runScripts:"dangerously", url:"https://x/#/" });
// dom.window 에서 S, commit, migrate, mdToHtml ... 를 꺼내 검사
```

개발 당시 돌린 검사 범위:

| 범주 | 검사 내용 |
|---|---|
| 저장·영속성 (15가지) | 저장 실패 감지, 5초 재시도, `BAK` 복구, 다른 탭 충돌, `flush` 경로 |
| **기록 보존 (29가지)** | 템플릿 교체·복제, 단 수 변경, 백업 왕복, 가져오기 후 내용이 전부 남는지 |
| 일반 점검 (9개 범주) | HTML 주입, **모든 `data-act` 버튼을 하나씩 눌러보기**, 2400개 스트레스 |
| 내부 ID 누출 | 센티넬 ID를 데이터에 심고 UI·PDF·검색·집계에 나오지 않는지 |
| 한글 IME | 조합 중 Enter, 목록 이어쓰기, 커서 위치 |
| 마크다운 | 제목 접기, 구분점 단계, 번호 매김, 줄바꿈 보존, 서식 겹침 |

새 기능을 넣을 때 최소한 **기록 보존**과 **모든 `data-act` 버튼 클릭**은 다시 돌린다.
이 두 가지가 과거에 가장 많이 잡아냈다.

### 레이아웃·PDF는 실제로 렌더해서 재야 한다

jsdom은 레이아웃을 계산하지 않는다. 좌표·픽셀이 걸린 문제는 헤드리스 Chromium을 쓴다.

```bash
chromium --headless --dump-dom        work.html   # 실제 DOM
chromium --headless --print-to-pdf=o.pdf work.html   # PDF
chromium --headless --screenshot --window-size=1024,1366 work.html
pdftotext o.pdf -                                 # 글자가 다 들어갔는지
pdftoppm -png o.pdf page                          # 픽셀 확인 (PIL/numpy로 색 세기)
```

이 방식으로 확인한 것들: 하이라이트 노란 픽셀 1244개가 인쇄에 살아남음, 160/160줄이
페이지 넘김에서 유실되지 않음, 폴더 타일 실제 크기.

### 테스트가 틀렸을 가능성을 먼저 본다

테스트 실패 중 몇 건은 **테스트의 기대값이 틀린 것**이었다 (커서 오프셋, 구분점 기본값).
앱 코드를 고치기 전에 기대값이 맞는지 확인한다.

---

## 10. 작업 체크리스트

고치고 끝내기 전에 훑는다.

```
[ ] 기록 보존에 영향이 있나? → 템플릿 교체·단 변경·백업 왕복 후 내용이 남는지 확인
[ ] 회차 객체에 새 키를 넣었나? → LESSON_META 에 추가, 내부 ID 누출 확인
[ ] 새 설정 키를 넣었나? → migrate() 에 기본값 채우기
[ ] 새 data-act 를 넣었나? → 섹션 11 switch 에 추가, 버튼을 실제로 눌러보기
[ ] 새 화면/필드를 넣었나? → PDF 경로(섹션 8)에도 반영
[ ] 입력칸을 건드렸나? → 입력 중 높이 변경 없음, autocorrect="on", editTA() 사용
[ ] 아이콘을 넣었나? → SVG 크기 CSS 지정
[ ] 사용자 글을 표시하나? → slice() 로 안 자르고 CSS ellipsis
[ ] 삭제 기능인가? → toastUndo() 로 8초 되돌리기
[ ] 아이패드 폭에서 확인했나? → 1024×1366 스크린샷
[ ] 버전 3곳 동기화 → APP_VERSION / CHANGELOG / version.json
[ ] HISTORY.md 에 원인과 고친 방법 기록
[ ] 커밋하고 푸시 + main 반영 → 묻지 않고 매번 (아래)
```

### 업데이트가 끝나면 매번 커밋·푸시하고 main에도 반영한다

사용자 요청(2026-11-07): *"앞으로 업데이트 할때마다 푸시해줘"*,
*"업데이트 될때마다 main에도 바로 반영해줘"*

- 고치고 검증까지 끝나면 **따로 묻지 않고** 커밋하고 그 세션의 작업 브랜치에 푸시한다.
  문서(CLAUDE.md·HISTORY.md)만 고친 경우도 같다.
- 이어서 **`main`에도 바로 반영한다** (PR 없이). GitHub Pages가 `main`에서 배포되므로
  이렇게 해야 아이패드 앱에 바로 들어간다.
  - 먼저 `git fetch origin main` 으로 `main`이 작업 브랜치의 조상인지 본다.
    조상이면 `git push origin <작업브랜치>:main` (fast-forward) 로 올린다.
  - `main`에 작업 브랜치에 없는 커밋이 있으면(사용자가 GitHub에서 직접 올린 파일 등)
    **강제 푸시하지 않는다.** `main`을 작업 브랜치에 병합해 충돌을 정리하고, 다시 검증한 뒤 올린다.
    사용자가 올린 것을 덮어쓰면 안 된다.
  - 올린 뒤 `origin/main`이 방금 커밋을 가리키는지 확인한다.
- **검증을 통과하지 못한 것은 `main`에 올리지 않는다.** `main`은 사용자가 실제로 쓰는 앱이다.
  버전 3곳(§2)이 맞지 않은 채 올리면 업데이트 알림이 틀어진다.
- 커밋에는 index.html·version.json·CLAUDE.md·HISTORY.md처럼 저장소 파일만 넣는다.
  테스트 스크립트·스크린샷은 넣지 않는다 (§9).

---

## 11. 하지 말 것

- 빌드 도구·프레임워크·npm 의존성 추가
- 파일 쪼개기 (단일 HTML 유지)
- 다크모드 (흰 배경 고정)
- 데이터를 외부로 보내기 — 서버 전송·분석·외부 API 호출. 노트는 기기 안에만 있다.
  업데이트 확인용 `version.json` 읽기가 유일한 네트워크 요청이다.
- `localStorage` 키 이름 변경 (`lecture-notes.v1`) — 기존 사용자의 데이터가 사라진다.
  구조를 바꿀 땐 `schema` 번호를 올리고 `migrate()`에서 변환한다.
- 필드 키(`f_...`) 재생성
- 사용자 입력을 `innerHTML`에 그대로 넣기 — 반드시 `esc()`를 통과시킨다.
  마크다운 렌더도 `mdToHtml(esc(val), ...)` 순서다.
