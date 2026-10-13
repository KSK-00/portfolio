# TradingView 기반 주문 자동화

개인 프로젝트의 구현 내용과 기존 실행 기록을 정리한 한국어 요약입니다.

[프로젝트 상세 페이지](https://portfolio-643.pages.dev/kr/trading/)

## 구현 내용

- TradingView Webhook으로 수신한 신호를 주문 API에 연결하는 개인 프로젝트입니다.
- 입력 확인과 주문 실행 역할을 분리하고 주문 요청 이후의 결과를 확인합니다.
- 브로커 응답과 주문·체결 상태를 구분해 기록합니다.
- 처리 결과를 로그와 Telegram 알림으로 확인할 수 있도록 구성했습니다.
- 투자 수익률이 아니라 API 연동, 상태 관리와 실행 확인의 구현 경험을 정리했습니다.

## 기술 활용

Java · Spring Boot · REST API/Webhook · 브로커 API · Telegram API · AWS EC2/Linux · Git/GitHub

## 실행 자료

기존 주문 연동 GIF와 한국투자증권·Upbit 실행 결과 화면. 상세 페이지에서 원본 비율로 확인하거나 확대할 수 있습니다. 이미지는 과거 실행 기록이며 실시간 서버 상태가 아닙니다.

## 공개 자료

[기존 일본어 README](https://github.com/KSK-00/portfolio/blob/main/jp/README.md)

운영 소스·인증 정보·개인정보는 포함하지 않았습니다.
