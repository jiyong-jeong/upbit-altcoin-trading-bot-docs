# 아키텍처

## 모듈 분해

```
trading-bot/
├── 거래소 클라이언트 래퍼
│   ├── myUpbit.py         pyupbit 위의 헬퍼 (잔고 조회, 주문, 캔들 등)
│   └── myBinance.py       ccxt 위의 헬퍼
│
├── 자동 매매 엔트리포인트
│   ├── upbit_best_bot.py        Upbit 전략 봇 (포트폴리오 운용)
│   ├── binance_auto_bot.py      Binance 자동 매수/매도
│   └── binance_auto_grid.py     Binance 그리드 전략
│
├── 키 / 보안
│   ├── ende_key.py        암복호화 키 (자체 생성)
│   └── my_key.py          암호화된 API 액세스/시크릿 키
│
├── 알림
│   └── line_alert.py      Line Notify 로 매매 결과 push
│
└── 데이터 / 설정
    ├── UpbitTopCoinList.json   추적할 코인 목록
    ├── AltATypeCoin.json       단타 A타입 후보
    ├── AltBTypeCoin.json       단타 B타입 후보
    ├── DolPaCoin.json          ? (별도 카테고리)
    ├── MaUpDict.json           이동평균 상태 dict
    └── RevenueDict.json        수익 누적 dict (상태 저장)
```

## 포트폴리오 분류 (코드 주석에서 추출)

| 분류 | 비중 | 매매 정책 |
|---|---|---|
| 비트코인 | 25% | 장기 보유 / 비중 유지 |
| 이더리움 | 10% | 장기 보유 |
| 알트 베스트 (장기) | 25% | 펀더멘털 기반 장기 보유 |
| 알트 단타 A | 8% | **상승장에서만 매수** |
| 알트 단타 B | 8% | 상승/하락장 모두 매수 |
| (나머지) | 24% | 현금/리저브 |

## A/B 타입의 의미

- **A타입 (상승장 매수)** — 가격이 상승 추세일 때만 신규 진입. 추세 추종.
- **B타입 (상시 매수)** — 추세와 무관하게 dip-buy. 평단가 낮추기 (DCA) 성격.

같은 단타 자금을 두 전략에 분산 → 한 쪽이 안 좋을 때 다른 쪽이 보전.

## 상태 저장 (JSON 파일)

봇이 stateless 가 아니라 **로컬 JSON 파일에 상태 누적**:
- `MaUpDict.json` — 각 코인의 이동평균 상황 (상승/하락 추세)
- `RevenueDict.json` — 종목별 실현 수익
- `AltATypeCoin.json`, `AltBTypeCoin.json` — 후보 목록 (수동 또는 자동 갱신)

→ 봇 재시작 시 상태 복구. 단, 두 instance 동시 실행하면 파일 race.

## 보안 / 키 관리

```
사용자가 한 번 ende_key.py 생성 (랜덤 시드)
   │
   ▼
업비트/바이낸스 API 키를 simpleEnDecrypt.encrypt() 로 암호화
   │
   ▼
my_key.py 에 암호화된 문자열 저장 (소스에 평문 키 X)
   │
   ▼
봇 시작 시 ende_key + my_key 로 복호화 → 메모리에 평문 키 → 거래소 호출
```

`my_key.py` 는 일종의 secrets 파일. **반드시 .gitignore** + ende_key 와 분리 보관.

## 알림 흐름

```
매매 실행 → line_alert.SendMessage("매수 BTC 0.001 @ ...")
                 │
                 ▼
            Line Notify API (token 인증)
                 │
                 ▼
            사용자 LINE 푸시
```

Slack/Discord/Telegram 도 동일 패턴. Line 은 한국 사용자에게 친숙.

## Binance Grid 봇 (별도 엔트리)

`binance_auto_grid.py` 는 그리드 매매 전략:
- 가격 구간을 N개 격자로 분할
- 매 격자에서 buy limit + 한 단계 위에 sell limit
- 횡보장에서 박스권 양 끝 사이의 이익을 누적

전략 자체가 코드 골격을 정한다 — 그리드 봇은 일반 추세 추종 봇과 완전히 다른 entrypoint 가 자연.

## 스케줄러

봇 자체에 `while True: ... time.sleep(N)` 패턴. 별도 cron 없이 daemon 으로 실행 (systemd / nohup / supervisor / tmux).
