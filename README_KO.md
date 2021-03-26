# 오사카부 코로나19 대책 사이트 

![](https://github.com/codeforosaka/covid19/workflows/production%20deploy/badge.svg)

[오사카부 코로나19 대책 사이트](https://covid19-osaka.info/)

## 기여방법
Issues 에 있는 여러 안건에 협력해주시면 감사하겠습니다.

자세한 내용은 다음의 [기여방법](./.github/CONTRIBUTING.md)을 참고해주세요.

## 행동강령
자세한 내용은 다음의 [사이트 구축 행동강령](./.github/CODE_OF_CONDUCT.md)을 참고해주세요.

## 라이선스
이 소프트웨어는 [MITライセンス](./LICENSE.txt) 를 따르고 있습니다/

## 이 사이트의 근본

### 도쿄도 코로나19 대책 사이트
[사이트 링크](https://stopcovid19.metro.tokyo.lg.jp/)

[Github 링크](https://github.com/tokyo-metropolitan-gov/covid19)

## 개발자를 위한 개발 정보

### data.json의 설명
* JSON형식의 데이터입니다.
* 지금은 수작업으로 수정하고 있습니다.（오사카부의 관계자께서 Pull Request 로 수정 해 주시는 경우가 많습니다.）

#### patients
* 감염자수
* date：최종 갱신일.
* data -> リリース日：배포일. 발표된 날짜（마지막의 `T08:00:00.000Z`은 고정）
* data -> 曜日：요일. 다른행과 동일하게 입력.
* data -> 居住地：주거지. 시,구,읍,촌까지.
* data -> 年代：연령대. ~~대(〜〜代) 까지 입력.
* data -> 性別：성별. 남성 또는 여성 입력. (男性 or 女性)
* data -> 退院：퇴원. 퇴원한 경우는「○」, 사망 한 경우는 「死亡」, 그외에는「（空白）」
* data -> date：위의 배포일과 동일하게 입력.（포멧은 다르지만..）

#### patients_summary
* 일별 감염자수
* date：최종 갱신일
* data -> 日付：날짜. 발표한 날짜（마지막의 `T08:00:00.000Z`은 고정）
* data -> 小計：소계. 그 날의 감염자수（누계가 아니므로 주의）

#### inspections_summary
* 일별 검사건수
* date：최종 갱신일
* data -> 日付：날짜. 발표한 날짜（마지막의 `T08:00:00.000Z`은 고정）
* data -> 小計：소계. 그 날의 검사건수（누계가 아니므로 주의）

#### contacts1_summary
* 오사카부민 대상 상담창구로의 상담건수
* date：최종 갱신일
* data -> 日付：날짜. 발표한 날짜（마지막의 `T08:00:00.000Z`은 고정）
* data -> 小計：소계. 그 날의 상담건수（누계가 아니므로 주의）

#### contacts2_summary
* 코로나19 검진상담센터로의 상담건수
* date：최종 갱신일
* data -> 府管轄保健所：부관할보건소. 일별 오사카부 관할 보건소로의 상담건수（누계가 아니므로 주의）
* data -> 政令中核市保健所：정령중핵시보건소. 정령에 의거하는 시의 보건소로의 상담건수（누계가 아니므로 주의）
* labels：날짜

#### treated_summary
* 음성 확인（퇴원환자 누계）
* date：최종 갱신일
* data -> 퇴원일（마지막의 `T08:00:00.000Z`은 고정）
* data -> 小計：소계. 그날 퇴원 건수（누계가 아니므로 주의）

#### main_summary
* 상황 요약
* http://www.pref.osaka.lg.jp/iryo/osakakansensho/corona.html
* 상기 페이지의 숫자와 맞춤（수작업）

### 개발환경을 구축하는 방법

- 필요한 Node.js 버젼: 10.19.0이상

**yarn 을 사용할 경우**
``` bash
# install dependencies
$ yarn install

# serve with hot reload at localhost:3000
$ yarn dev
```

**docker compose 을 사용할 경우**
```bash
# serve with hot reload at localhost:3000
$ docker-compose up --build
```

### `Cannot find module ****` 라고 오류가 나는 경우

**yarn 을 사용할 경우**
```
$ yarn install
```

**docker compose 을 사용할 경우**
```bash
$ docker-compose run --rm app yarn install
```

### 스테이징 환경 및 운영 환경으로 반영 방법

`master` 브런치가 업데이트 되면, 자동적으로 `production` 브랜치의 HTML 파일이 빌드됩니다. 이후, 운영 환경 사이트 https://covid19-osaka.info/ 가 갱신됩니다.

`staging` 브런치는 오사카 코로나19 프로젝트에서는 운용하지 않습니다. develop에서 master로 Pull Request 해 주시기 바랍니다

`development` 브랜치가 업데이트 되면, 자동적으로 `dev-pages` 브랜치의 HTML 파일이 빌드됩니다. 이후, 개발용 사이트 https://dev-covid19-osaka.netlify.com/ 가 갱신됩니다.
