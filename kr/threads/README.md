# Threads 운영 자동화

개인 프로젝트의 구현 내용과 기존 실행 기록을 정리한 한국어 요약입니다.

[프로젝트 상세 페이지](https://portfolio-643.pages.dev/kr/threads/)

## 구현 내용

- AI가 생성한 후보를 검사한 뒤 조건을 통과한 게시물만 발행하도록 구성했습니다.
- 본문 길이·금칙어·언어 정책과 기존 게시 이력의 중복·유사도를 확인합니다.
- 유효한 후보가 없으면 재생성하거나 게시를 중단하고 처리 결과를 기록합니다.
- 생성한 답글에도 길이·금칙어·언어 검사를 적용해 발행을 제어합니다.
- 대시보드에서 예약 계획과 게시·검사 결과를 확인하고 Telegram으로 알림을 전달합니다.

## 기술 활용

Java 21 · Spring Boot · Meta Threads API · OpenAI API · Telegram Bot API · Linux · Git/GitHub

## 실행 자료

하루 3회 게시 계획이 표시된 기존 대시보드, 실행 GIF, 댓글·로그·일일 보고서·알림. 상세 페이지에서 원본 비율로 확인하거나 확대할 수 있습니다. 이미지는 과거 실행 기록이며 실시간 서버 상태가 아닙니다.

## 공개 자료

[기존 일본어 README](https://github.com/KSK-00/portfolio/blob/main/jp/threads/README.md)

운영 소스·인증 정보·개인정보는 포함하지 않았습니다.
