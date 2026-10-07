# TETRIX

인트로 영상 속 블록 친구들과 함께하는 웹 테트리스입니다. `index.html` 한 파일과 `video/intro.mp4`로 동작합니다.

- 타이틀 화면에서 **게임 시작**을 누르면 인트로 영상이 재생되고(건너뛰기 가능), 3·2·1 카운트다운 뒤 게임이 시작됩니다.
- 연속으로 줄을 지우면 **콤보**가 쌓이고, 그때마다 블록 친구들이 보드 옆으로 튀어나와 응원합니다. TETRIX(4줄 동시 삭제)와 퍼펙트 클리어 때도 나옵니다.
- 표준 회전 규칙(SRS), 홀드, NEXT 3개, 고스트 블록, 잠금 지연, 7-bag 순서를 지원합니다.
- 한국어 / English / 日本語 / 中文 4개 언어를 지원합니다.
- 배경음악은 Web Audio로 실시간 합성하며, 원래 버전의 멜로디를 그대로 사용합니다.

## 조작

| 키 | 동작 |
|---|---|
| ← → | 이동 |
| ↓ | 소프트 드롭 |
| Space | 하드 드롭 |
| ↑ / X | 회전 |
| Z / Ctrl | 반대 방향 회전 |
| C / Shift | 홀드 |
| P / Esc | 일시정지 |
| M | 음악 켜기/끄기 |

모바일에서는 화면 아래 버튼으로 조작합니다.

## 실행

`index.html`을 더블클릭해도 실행됩니다. 로컬 서버로 열려면 다음 명령을 사용합니다.

```bash
python -m http.server 8000
# http://localhost:8000
```

GitHub Pages로 배포할 때는 `index.html`과 `video/` 폴더를 저장소 최상위에 올리고 **Settings → Pages**에서 `main` 브랜치의 `/ (root)`를 선택합니다.

## 글로벌 순위표 (Firebase)

`index.html` 상단의 `FIREBASE_URL`에 `https://unlv-tet-default-rtdb.firebaseio.com`이 들어 있습니다.
현재 이 데이터베이스는 **읽기/쓰기가 거부(401)** 되는 상태입니다. 테스트 모드의 30일 기한이 끝난 것으로 보입니다.
이 상태에서도 게임은 정상 동작하며, 기록은 이 브라우저에만 저장됩니다.

모든 방문자가 같은 순위표를 보게 하려면 Firebase 콘솔 → Realtime Database → **규칙(Rules)** 에 아래 내용을 붙여넣고 **게시**하세요.

```json
{
  "rules": {
    "scores": {
      ".read": true,
      ".indexOn": ["score"],
      "$id": {
        ".write": "!data.exists()",
        ".validate": "newData.hasChildren(['initials', 'score', 'date']) && newData.child('initials').isString() && newData.child('initials').val().matches(/^[A-Z0-9]{1,3}$/) && newData.child('score').isNumber() && newData.child('score').val() >= 0 && newData.child('score').val() < 100000000 && newData.child('date').isString() && newData.child('date').val().length == 10 && (!newData.child('lines').exists() || newData.child('lines').isNumber())"
      }
    }
  }
}
```

- `"$id": { ".write": "!data.exists()" }`: 새 기록 추가만 허용합니다. 다른 사람의 기록을 덮어쓰거나 지울 수 없습니다.
- `.validate`: 이니셜(영문/숫자 3자)과 점수 형식이 맞지 않는 데이터를 거부합니다.
- `.indexOn`: 서버에서 점수순으로 상위 20개만 받아오게 해 줍니다.

참고로 게임 쪽에서도 받아온 데이터를 형식 검사한 뒤 텍스트로만 화면에 표시하므로, 누가 이상한 값을 넣어도 페이지에서 코드가 실행되지 않습니다.
다만 로그인 없는 캐주얼 순위표라서, 개발자 도구로 점수를 조작해 등록하는 것까지는 막을 수 없습니다.

## 파일 구조

```
index.html        게임 전체 (타이틀 · 인트로 · 게임 · 순위표 · 다국어 · 사운드)
video/intro.mp4   인트로 영상
```
