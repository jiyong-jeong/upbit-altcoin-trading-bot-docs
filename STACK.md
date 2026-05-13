# 스택

## Python 라이브러리

| 라이브러리 | 용도 |
|---|---|
| **pyupbit** | Upbit REST + websocket. 한국 거래소 표준 |
| **ccxt** | 통합 거래소 클라이언트. Binance, Bybit, OKX 등 100+ 거래소 |
| **pandas** | 캔들 데이터 분석, indicator 계산 |
| **requests / aiohttp** | 직접 HTTP 호출이 필요한 경우 |

## 알림

- **Line Notify** — 한국 사용자에게 친숙. token 기반 push (Line 2025년 이후 일부 서비스 종료 예정 — 대안 마련 권장)
- **Telegram Bot API** — 글로벌, 매우 안정적
- **Slack incoming webhook** — 팀 알림에 자연

## 데이터 / 상태 저장

| 방식 | 장점 | 단점 |
|---|---|---|
| JSON 파일 (현재 패턴) | 가장 단순 | race condition 위험 |
| SQLite | 트랜잭션 + 동시성 | 약간의 학습 부담 |
| PostgreSQL | 본격적 | 봇 한 대 운영에 과함 |

## 키 / 비밀 관리

- **자체 암복호화** (현재 패턴) — 학습용으로 OK. 두 키 파일 분리 보관 필수
- **OS keychain** (macOS Keychain, Linux Secret Service) — 더 권장
- **HashiCorp Vault / AWS Secrets Manager** — 본격적 운영
- **환경변수 + dotenv** — 가장 흔함

## 모니터링 / 운영

- **systemd unit** — Linux 서버에서 daemon 화
- **supervisor** — 다중 봇 관리
- **screen / tmux** — 학습 단계
- **Prometheus + Grafana** — 본격적 지표 시각화

## 백테스트 / 분석

- **backtesting.py** — Python 기반 backtest 프레임워크
- **freqtrade** — 봇 + backtest 통합 OSS
- **TradingView Pine Script** — 시각적 indicator 작성

## 거래소 별 특이사항

### Upbit (한국)
- KRW 마켓 (BTC 마켓도 있음)
- 출금/입금 권한이 거래 권한과 분리됨 — 봇은 거래 권한만
- 최소 주문 5,000 KRW
- 수수료 0.05% (테이커/메이커 동일)

### Binance (글로벌)
- 현물 + 선물 + 마진
- USDT, USDC, BUSD (deprecated) 마켓
- weight-based rate limit
- 수수료 0.1% (BNB 지불 시 0.075%)

## 대안 / 발전 방향

- **freqtrade** — OSS 트레이딩 봇 프레임워크. 전략을 strategy 클래스로 분리
- **Hummingbot** — 마켓 메이킹 / 차익 거래 특화
- **자체 deep learning 모델** — TensorFlow / PyTorch + LSTM 등. 학습 + 실거래 사이 검증 부담 큼

## 법적 / 세무 주의

- 한국: 가상자산 매매로 인한 소득은 2025년 이후 과세 (2024-12 시점 기준). 정책 변동 가능.
- API 키 leak 으로 인한 손실은 거래소 보상 대상 아닐 수 있음 — 본인 책임
