# 오늘의 단어 - Firebase 공유 기능

이 앱은 Firebase Realtime Database를 이용해 다음을 지원합니다:

- 사용자 이름을 브라우저 쿠키/로컬스토리지에 저장
- 현재 문제를 Firebase에 저장하여 모든 사용자가 동일한 문제를 풀도록 동기화
- 최초로 문제를 맞힌 사람과 맞춘 시각을 Firebase에 기록

## 설정

1. Firebase 콘솔에서 프로젝트를 생성합니다.
2. Realtime Database를 활성화합니다.
3. 읽기/쓰기 규칙을 테스트 모드로 설정하거나, 적절한 보안 규칙을 구성합니다.
4. 웹 앱 등록 후 제공되는 Firebase 구성 정보를 `index.html`의 `FIREBASE_CONFIG`에 입력합니다.

```js
const FIREBASE_CONFIG={
  apiKey:'YOUR_API_KEY',
  authDomain:'YOUR_PROJECT.firebaseapp.com',
  databaseURL:'https://YOUR_PROJECT.firebaseio.com',
  projectId:'YOUR_PROJECT_ID',
  storageBucket:'YOUR_PROJECT.appspot.com',
  messagingSenderId:'YOUR_SENDER_ID',
  appId:'YOUR_APP_ID'
};
```

## 실행

간단하게 로컬 서버에서 확인합니다.

```bash
cd /Users/nohowon/Documents/wordle/todayword
python3 -m http.server 8000
```

브라우저에서 `http://localhost:8000`을 엽니다.

## Firebase 데이터베이스 구조

이 앱은 Firebase Realtime Database에 다음 구조를 사용합니다.

```json
{
  "currentId": "-Mabc12345",
  "entries": {
    "-Mabc12345": {
      "word": "학교",
      "solved": false,
      "solvedBy": "",
      "solvedAt": "",
      "updatedAt": "2026-06-09T12:00:00.000Z"
    }
  }
}
```

### 각 필드 설명

- `currentId`: 현재 풀어야 할 문제의 식별자
- `entries`: 문제 히스토리를 담는 객체
- `word`: 문제 단어
- `solved`: 해당 문제의 첫 정답자가 나왔는지 여부
- `solvedBy`: 첫 정답을 맞힌 플레이어 이름
- `solvedAt`: 첫 정답 시각(ISO 문자열)
- `updatedAt`: 마지막으로 변경된 시각(ISO 문자열)

## 수작업으로 첫 단어 등록하기

첫 문제는 Firebase 콘솔에서 직접 등록해주시면 됩니다.
1. `/sharedWord` 경로 아래에 `currentId`와 `entries`를 생성합니다.
2. `entries`에 첫 단어를 가진 객체를 추가합니다.
3. `currentId`에 해당 entry 키를 설정합니다.

예:

```json
{
  "currentId": "-Mabc12345",
  "entries": {
    "-Mabc12345": {
      "word": "학교",
      "solved": false,
      "solvedBy": "",
      "solvedAt": "",
      "updatedAt": "2026-06-09T12:00:00.000Z"
    }
  }
}
```

## 동작 방식

- 페이지 로드 시 Firebase에서 `/sharedWord`를 읽습니다.
- 현재 문제가 없거나 `solved: true`면 앱이 새로운 단어를 랜덤으로 생성해 새로운 entry를 등록합니다.
- 사용자가 문제를 맞히면 첫 정답자 정보가 `solved`, `solvedBy`, `solvedAt`, `updatedAt`에 기록됩니다.
- 첫 정답자가 된 클라이언트는 즉시 다음 문제를 생성하여 Firebase에 새 row를 추가합니다.
- 다른 사용자는 실시간으로 공유 문제 상태를 갱신합니다.
