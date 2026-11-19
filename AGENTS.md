# 웹북허브 제출 형식 규칙 (AI 도구용)

당신(AI)은 작가를 도와 이 저장소를 **웹북허브 제출 형식 `webbook-submission/2`**에 맞춥니다.
이 규칙을 벗어난 파일·필드는 검사에서 거부되니, 아래에 적힌 것만 만듭니다.

## 1. 저장소 구조

```text
book.json            필수. 책 한 권의 모든 글과 그림 연결
assets/<이름>.<확장자>  선택. 그림·영상·소리. 하위 폴더 금지
README.md, AGENTS.md, LICENSE, .gitignore, .gitattributes, .github/   검사에서 무시
```

- 위 목록 밖의 파일·폴더를 만들지 않습니다(`index.html`, `.js`, `.css`, `pages/`, `src/`, `package.json` 등 모두 금지).
- `manifest.json`은 만들지 않습니다. 운영자 도구가 자동으로 만듭니다.

## 2. assets 파일 이름

- 정규식 `^[a-z][a-z0-9_-]{0,63}\.(png|jpg|jpeg|webp|gif|mp4|webm|mov|mp3|m4a|aac|ogg|wav)$`
- **확장자를 뺀 이름이 그 파일의 ID**입니다. `assets/page-03.png` → `"page-03"`.
- 확장자만 다르고 이름이 같은 파일 두 개(`cover.png`, `cover.jpg`)는 금지입니다.
- 그림 PNG·JPG·WebP는 한 장 8MB 이하, 영상은 60초·50MB 이하입니다.

## 3. book.json

엄격한 JSON(UTF-8, 주석·끝 쉼표·중복 키 금지)입니다. 최상위 키는 아래만 씁니다.

| 키 | 필수 | 값 |
|---|---|---|
| `title` | 필수 | 책 제목, 160자 이하 |
| `subtitle` | 선택 | 240자 이하 |
| `description` | 선택 | 책 소개, 4000자 이하 |
| `genre` | 필수 | `picturebook` · `fiction` · `learning` · `essay` · `reference` 중 하나 |
| `aspectRatio` | 필수 | `3:4`(휴대폰 세로, 권장) · `4:3` · `16:9` · `9:16` |
| `coverAssetId` | 선택 | 표지 그림 ID (assets 파일 이름) |
| `credits` | 필수 | `{ "author": "작가 이름", "tools": ["실제로 쓴 도구", …] }` — tools 1개 이상 |
| `pages` | 필수 | 쪽 배열, 1~500쪽 |
| `glossary` | 선택 | `[{ "id", "term", "definition", "pages": [쪽 id…] }]` |

쓰지 않는 키: `projectId`, `owner`, `status`, `offer`, `publishedBuildId`, `approvalHash` 등 운영 필드. `prj_`, `pg_`, `ast_`로 시작하는 ID도 금지입니다.

### 쪽(page)

```json
{
  "id": "forest",
  "title": "숲으로",
  "body": "본문. 줄바꿈은 \n",
  "layout": "split",
  "imageAssetId": "forest",
  "imageAlt": "그림을 설명하는 한 문장",
  "interactions": []
}
```

- `id`: 영문 소문자로 시작, `[a-z0-9_-]`, 64자 이하, 책 안에서 겹치지 않게.
- `layout`: `split`(그림과 글 나눔) · `bleed`(그림 위에 글) · `text`(글만). 그림이 없는 쪽은 `text`.
- 그림이 있으면 `imageAlt`를 꼭 씁니다(500자 이하).
- `title` 240자, `body` 30,000자 이하. 쪽마다 인터랙션 8개 이하(2개 이하 권장).

### 인터랙션 (기본 3종)

```json
{ "id": "hint", "type": "reveal", "title": "작은 쪽지", "label": "쪽지 펴 보기", "body": "숨은 글" }
```
```json
{ "id": "way", "type": "choice", "title": "어느 길로 갈까요?",
  "options": [ { "label": "언덕 길", "goTo": "hill" }, { "label": "시냇물 길" } ] }
```
```json
{ "id": "hill-ending", "type": "passage", "body": "앞에서 언덕 길을 고른 독자에게만 보이는 글",
  "showWhen": { "page": "choose", "interaction": "way", "picked": [0] } }
```

- `id`는 쪽 안에서 겹치지 않게, 영문 소문자로 시작.
- `choice.options`는 2~6개. `goTo`는 이동할 쪽 `id`(선택).
- `showWhen.picked`는 0부터 세는 보기 번호 배열.
- 퀴즈·자물쇠·그림 속 찾기·쪽 영상·눌러 보기 반응·튀어나오기·번역·코드형 미니게임까지, 운영자 책에 쓰는 고급 기능은 모두 이 형식으로 받습니다. 직접 JSON을 쓰기보다 작가 엔진의 `iwan-webbook-mcp`로 만들고 `author-sync save`로 저장하면 형식이 맞게 담깁니다. 미니게임은 독자 화면에서 격리된 칸 안에서만 돌아갑니다.

## 4. 반드시 지킬 내용 규칙

1. **AI 생성 표기를 쓰지 않습니다.** 본문·소개에 "AI로 만들었습니다" 같은 문장을 넣지 않습니다. 사용한 도구는 `credits.tools`에만 적고, 책 마지막 쪽에 자동으로 표시됩니다.
2. **실제로 쓴 도구만** `credits.tools`에 적습니다.
3. **권리:** 작가가 직접 만들었거나 사용 허락을 받은 글·그림·소리만 씁니다. 남의 캐릭터·사진·음악을 가져오지 않습니다.
4. 실존 인물을 사칭하거나 성적·혐오 표현을 넣지 않습니다.
5. 그림마다 `imageAlt`를 씁니다.

## 5. 끝내기 전 스스로 확인

- [ ] 루트에 `book.json`과 `assets/`(+ 무시되는 안내 파일)만 있다.
- [ ] `book.json`이 엄격한 JSON으로 파싱된다.
- [ ] 모든 `imageAssetId`·`coverAssetId`가 assets 안 파일 이름(확장자 뺀)과 정확히 같다.
- [ ] 모든 `goTo`·`showWhen.page`가 실제 쪽 `id`를 가리킨다.
- [ ] `credits.author`와 `credits.tools`(1개 이상)가 있다.
- [ ] 그림이 있는 쪽마다 `imageAlt`가 있다.

이 저장소의 `book.json`(예시 3쪽)이 올바른 형태의 예입니다. 같은 모양을 따라 만듭니다.

## 6. 책을 잘 만드는 요령

형식은 위 규칙이 정하고, 이야기·인터랙션·그림을 어떻게 만들지는 **웹북허브 작가 스킬**이 정합니다. 책 내용을 만들거나 고치기 전에 읽습니다.

- 깔았다면: `~/WebbookHub/skill/webbookhub-author/SKILL.md` (Claude Code는 `~/.claude/skills/webbookhub-author/`에서 저절로 불립니다)
- 안 깔았다면: https://webbookhub.com/author/skill/webbookhub-author/SKILL.md 를 읽고, 까는 법은 https://webbookhub.com/author/agent.md 의 '작가 스킬 깔기'
