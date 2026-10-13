# LINE 문의·예약 자동화

개인 프로젝트의 구현 내용과 기존 실행 기록을 정리한 한국어 요약입니다.

[프로젝트 상세 페이지](https://portfolio-643.pages.dev/kr/LINE/)

## 구현 내용

- LINE Webhook의 서명과 중복 이벤트를 확인하고 대화 상태에 따라 예약 처리를 진행합니다.
- 예약 등록·변경·취소를 Google Calendar에 반영하고 상담 내용을 Sheets에 기록합니다.
- AI의 의도 파악과 애플리케이션의 업무 조건 검사·실행을 분리했습니다.
- 확정 예약을 확인해 Telegram 알림을 전송하며 중복 알림을 제한합니다.
- 정상·누락·잘못된 서명, 예약 알림 조건과 실패 재시도를 다루는 테스트를 작성했습니다.

## 기술 활용

Java 21 · Spring Boot · LINE Messaging API · OpenAI API · Google Calendar/Sheets · Telegram API · AWS/Ubuntu

## 실행 자료

문의·예약 등록·기록 공유와 예약 변경·취소를 기록한 기존 GIF. 상세 페이지에서 원본 비율로 확인하거나 확대할 수 있습니다. 이미지는 과거 실행 기록이며 실시간 서버 상태가 아닙니다.

## 공개 자료

[기존 일본어 README](https://github.com/KSK-00/portfolio/blob/main/jp/LINE/Readme.md)

운영 소스·인증 정보·개인정보는 포함하지 않았습니다.
