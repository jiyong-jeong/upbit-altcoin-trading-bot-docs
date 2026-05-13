# 배운 점

> 관찰자 시점. 개인용 암호화폐 자동 매매 봇 운영에서 일반적으로 마주치는 함정.

## API 키 보안 함정

### 평문 키를 코드/git 에 노출
가장 흔하고 가장 비싼 사고. 한 번 leak 되면 봇 운영자뿐 아니라 누구나 거래 가능 → 잔고 즉시 전송 가능.

→ 권장:
- API 키는 환경변수 또는 별도 비공유 파일 (`.gitignore`)
- IP whitelist 설정 (거래소가 지원)
- 출금 권한 절대 X (오직 거래 권한만)

### `ende_key` 와 `my_key` 둘 다 git 에 들어가면 의미 없음
자체 암복호화는 두 파일이 분리되어야 의미가 있다. 한 repo 에 둘 다 들어가면 평문 저장과 동등.

→ 적어도 `my_key.py` (암호화된 키 저장 파일) 은 .gitignore. `ende_key.py` (키 자체) 도.

### IP whitelist 미설정
Upbit/Binance 모두 API 키별 IP whitelist 옵션 제공. 봇이 도는 서버 IP 만 허용 → 키 leak 되어도 외부에서 사용 불가.

## 거래소 API rate limit 함정

### 분당 호출 수 제한
- Upbit: 초당 8-10 회 정도 (REST). websocket 은 별도
- Binance: weight 단위. 분당 1200 weight

순진하게 1초마다 모든 코인의 캔들 조회하면 100코인 * 1초 = 즉시 limit hit → 429.

→ 권장:
- 캔들/잔고는 캐시 (수 초~수십 초)
- websocket 으로 가격 stream (REST 폴링 줄임)
- 분산 backoff

### Order 의 멱등성
같은 주문을 두 번 보내면 두 번 체결 가능. 네트워크 timeout 후 retry 가 위험.

→ 권장: clientOrderId 활용 (Binance), Upbit 는 사전에 잔고 변화 확인 후에만 재시도.

## 전략 자체의 함정

### "상승장에서만 매수" 의 정의
이동평균 골든크로스? RSI > 50? 직전 24h 수익률 > 0? — 정의 따라 결과가 완전히 다름. 단순 backtest 로는 우열 가리기 어려움.

### 백테스트 vs 실거래 괴리
- 슬리피지 (호가 두께)
- 수수료 (Upbit 0.05%, Binance 0.1% 등)
- 체결 지연 (시장가 vs 지정가)

백테스트에서 +10% 도 실거래에서는 본전 또는 손실 가능.

### overtrade
빈번한 매매는 수수료가 누적 손실. 100회 매매에 매번 0.1% 면 10% 가 그냥 사라짐 (왕복은 0.2% 씩 × 50회 왕복 = 10%).

→ 권장: 최소 hold time, 최소 수익률 임계치.

## 상태 저장 파일의 함정

### JSON 파일 race condition
봇이 두 instance 동시 실행되면 `MaUpDict.json` 을 동시 write → corrupt.

→ 권장: 파일 lock (`fcntl` / `flock`) 또는 SQLite.

### 봇 crash 시 상태 inconsistent
write 도중 crash → 빈 파일 / 부분 JSON. 다음 시작 시 parse 실패 → 상태 손실.

→ 권장: atomic write 패턴 (`temp file → rename`).

## 알림 / 모니터링

### Line Notify (또는 모든 무료 push) 의 rate limit
빈번한 매매 시 알림이 throttle. 중요 매매가 누락 알림으로 묻힘.

→ 권장: 알림 우선순위 분리. 매매는 즉시, 모니터링 상태는 5분 summary.

### 손익 모니터링 부재
"수익률이 -10% 이하 되면 자동 손절" 같은 가드 없으면 한 번의 폭락이 큰 손실.

## 일반화 가능한 교훈

1. **API 키 보안이 첫째 그리고 가장 비싼 문제** — IP whitelist + 권한 최소화 + 출금 X
2. **rate limit + 멱등성** — 두 가지가 실거래 운영의 90%
3. **백테스트와 실거래의 괴리를 의식** — 수수료 + 슬리피지가 결과를 뒤집을 수 있음
4. **상태 저장은 atomic write 또는 작은 DB** — JSON 파일은 단일 instance 가정에서만 안전

---

> ⚠️ 본 노트는 코드 구조 관찰일 뿐이며, 어떤 전략의 수익성도 보장하지 않는다.
> TODO: 본 봇의 실제 운영 결과 / 회고 추가.
