# RoleLens — 내 조건으로 읽는 채용공고

반복해서 공고를 읽는 작업을 후보 비교·필수 조건·미확인 항목 확인으로 정리한 개인 프로젝트입니다. 공개명은 RoleLens이며 HUNTERY·Daily Job Hunter는 기존 제품 UI·알림에 남아 있는 이름입니다.

## 사용 흐름

개인 조건으로 후보 정리 → 추천·평가 현황을 Telegram으로 확인 → 코드로 권한 확인 → 개인 웹에서 근거·관심 공고·검토 상태 확인.

사람인·잡코리아 정보와 개인 조건을 바탕으로 검토를 보조합니다. 필수 조건과 우대 조건을 구분하며 미확인 정보는 확정 사실로 바꾸지 않습니다. 자동 지원이나 이력서 제출은 하지 않습니다.

## 실제 화면

웹 촬영 2026.11.04 · 기존 Telegram 수신 화면 촬영 2026.11.05. 회사·공고·위치·개인 상태와 추천 이유·접속 정보는 마스킹하거나 제외했습니다. 아래는 공개용으로 검수한 실제 캡처입니다. 화면 수치·점수는 촬영 당시 표시로 합격 확률이나 성과가 아닙니다.

![대시보드](https://portfolio-643.pages.dev/kr/assets/rolelens/live-dashboard.png)
![추천 비교](https://portfolio-643.pages.dev/kr/assets/rolelens/live-recommendations.png)
![필수 조건](https://portfolio-643.pages.dev/kr/assets/rolelens/live-requirements.png)
![기존 평가 현황 알림 발췌](https://portfolio-643.pages.dev/kr/assets/rolelens/live-telegram-evaluation.png)
![알림에서 웹으로 이동하는 버튼](https://portfolio-643.pages.dev/kr/assets/rolelens/live-telegram-web-actions.png)

[화면과 공개용 원본 보기](https://portfolio-643.pages.dev/kr/rolelens/#screens). 관리 패널과 로컬 평가 데모는 합성 예시로 별도 표시합니다.

## 사용 중 바꾼 점과 확인 범위

- 재게시 ID가 달라 중복되는 공고를 핵심 조건으로 묶었습니다. **2026.11.03 데이터 정리 기록:** 초과 중복 18건 제거, 2,675 → 2,657건. 기존 추천 131건의 ID와 개인 기록 보존.
- 처음 알리는 추천과 보유 추천을 구분했습니다. **2026.11.03 운영 데이터 복사본 시험:** 같은 상태 재실행 시 추가 추천 알림 0건. 실제 Telegram 발송 시험은 아닙니다.
- 같은 데이터를 반복 처리하던 부담을 정리하고 평가 대기 처리·보고서 종료를 확인했습니다. **2026.11.03 저장 상태 복사본 시험**이며 실제 정기 전체 검색 완료 검증과 구분합니다.

## 담당 역할과 구성

문제 정의, 개인 조건과 사용 흐름 설정, AI 개발 도구를 활용한 제작, 화면·기록 확인, 배포·운영 개선을 담당했습니다. Python, HTML/CSS/JavaScript, Telegram, AWS, HTTPS를 사용합니다. 운영 소스와 개인 설정은 공개하지 않습니다.

## 실행 안내

기존 Python 모듈 진입점은 다음과 같습니다. 공개명 변경을 위해 실행 경로를 바꾸지 않았습니다.

```sh
python -m employment_hunter --help
```

## 제약

외부 페이지 형식·접근 제한과 API 응답에 영향을 받습니다. 미확인 조건과 최종 지원 여부는 사람이 판단합니다. 2026.11.03 점검 기록의 Telegram 수신 충돌과 정기 전체 검색 결과는 웹·기존 알림 캡처만으로 해결 완료를 판정하지 않습니다.

[한국어 포트폴리오](https://portfolio-643.pages.dev/kr/) · [日本語](https://portfolio-643.pages.dev/jp/rolelens/)

연락처: [len007070@gmail.com](mailto:len007070@gmail.com)
