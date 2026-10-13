# 첫눈이 오기 전에

텍스트 기반 연애 시뮬레이션 웹 게임입니다. 3월 첫 만남부터 12월 첫눈 내리는 밤의 고백까지, 6개 이벤트와 13번의 선택으로 진행됩니다.

- 게임은 `index.html` 하나이고 빌드 과정이 없습니다. `worker/`는 Claude API 연결용 선택 사항입니다.
- 정해진 선택지를 고르거나 직접 말을 입력할 수 있습니다.
- 상대 성격(다정 / 차분 / 장난기)에 따라 호감도가 달라지고, 최종 호감도로 엔딩이 갈립니다.

## 엔딩 기준

| 호감도 | 엔딩 |
| --- | --- |
| 90 이상 | 완벽한 엔딩 |
| 70 ~ 89 | 연인 엔딩 |
| 45 ~ 69 | 열린 결말 |
| 44 이하 | 아쉬운 엔딩 |

## GitHub Pages로 배포하기

1. 새 저장소를 만들고 `index.html`, `README.md`를 올립니다.
2. 저장소의 Settings > Pages에서 Source를 `Deploy from a branch`, Branch를 `main` / `(root)`로 선택합니다.
3. 잠시 뒤 `https://<사용자명>.github.io/<저장소명>/` 주소로 접속할 수 있습니다.

로컬에서는 `index.html`을 브라우저로 바로 열면 됩니다.

## 직접 입력 반응에 Claude API 연결하기

기본 상태(`API_URL`이 빈 값)에서는 직접 입력한 말을 키워드로 읽는 기본 반응으로 진행합니다. Claude 앱 안에서 열면 별도 설정 없이 Claude가 즉석 반응을 만들어 줍니다.

GitHub Pages에서도 Claude 반응을 쓰려면 API 키를 숨겨 주는 중계 서버가 필요합니다. 키를 `index.html`에 넣으면 누구나 훔쳐 쓸 수 있으므로 절대 넣지 마세요. 이 저장소의 `worker/`는 무료로 쓸 수 있는 Cloudflare Worker 예시입니다.

1. [Anthropic Console](https://platform.claude.com)에서 API 키를 만들고, 사용 한도(spend limit)를 낮게 걸어 둡니다.
2. `worker/wrangler.toml`의 `ALLOWED_ORIGIN`에서 `<사용자명>`을 본인 GitHub 사용자명으로 바꿉니다.
3. `worker/` 폴더에서 다음을 실행합니다.

   ```bash
   npx wrangler login
   npx wrangler secret put ANTHROPIC_API_KEY   # 키를 붙여 넣기
   npx wrangler deploy
   ```

4. 배포가 끝나면 `https://first-snow-proxy.<계정>.workers.dev` 같은 주소가 나옵니다. 이 주소를 `index.html`의 `var API_URL="";` 안에 넣고 다시 커밋합니다.

주의할 점은 다음과 같습니다.

- `ALLOWED_ORIGIN` 검사는 다른 사이트의 브라우저 호출을 막을 뿐, 직접 요청을 보내는 사람까지 막지는 못합니다. Cloudflare의 Rate limiting 규칙을 걸고 Anthropic 사용 한도를 낮게 유지하세요.
- 기본 모델은 빠르고 저렴한 `claude-haiku-5-5`입니다. 대사 품질을 높이려면 `wrangler.toml`의 `MODEL`을 `claude-sonnet-5-5`로 바꿉니다.
- 중계 서버가 실패하거나 한도에 걸리면 게임은 자동으로 기본 반응으로 이어집니다.
- API 사용량은 키 소유자에게 과금됩니다. 자세한 내용은 [Claude API 문서](https://platform.claude.com/docs/en/api/messages)를 참고하세요.

## 내용 수정하기

`index.html` 안의 `<script>`에서 다음을 고치면 됩니다.

- `EVENTS`: 이벤트와 장면, 선택지, 상대의 대사
- `PERS`: 성격별 호감도 가중치와 표정 묘사
- `SETTINGS`: 만나는 곳별 첫 만남 장면
- `ENDINGS`: 엔딩 대사와 문구
