# Daily Stock Brief

매일 아침 한국·미국 시장에서 1~4주 스윙투자 관점으로 볼 만한 종목을 **시장별 최대 1개** 골라 Slack에 게시한다.

사람의 사전 검토 없이 게시되므로 **선정 수보다 정확성을 우선**한다. 조건을 충족하는 종목이 없으면 그 시장은 선정하지 않는다.

**지침 검토·수정 요청에서는 이 파일만 다룬다. 종목 조사·브리핑 작성·Slack 게시를 시작하지 않는다.**

## 기준 시각

- 실행을 시작한 KST 날짜를 **브리핑 날짜**로, 그날 07:00 KST를 **기준 시각**으로 고정한다(`TZ=Asia/Seoul date +%F`).
- 신호는 기준 시각 직전 **7일** 안에 발생·발행된 것만 쓴다.
- 종가는 기준 시각 직전 거래일 종가를 쓴다.

## 선정 기준

| 시장 | 대상 |
|---|---|
| 🇰🇷 한국 | 유가증권·코스닥 |
| 🇺🇸 미국 | NYSE·Nasdaq |

아래 다섯 조건을 **모두** 확인한 종목만 후보가 된다. 하나라도 확인하지 못하면 제외한다.

1. **긍정 신호**: 최근 7일 안에 다음 중 하나 이상이 있다.
   - 실적이 컨센서스를 상회
   - 회사의 가이던스 상향
   - 증권사 목표주가·투자의견 상향
   - 수주·공급계약 공시
   - 자사주 매입·소각 결정
   - 배당 확대 결정
2. **저평가**: PER(직전 종가 ÷ 최근 4분기 실제 EPS)이 동종업계 중앙값보다 낮다. 아래 [저평가 조회](#저평가-조회) 결과로 판정한다.
3. **실적**: 최근 분기 영업이익이 양수다.
4. **위험 없음 확인**: 최근 30일 안에 실적 하회·가이던스 하향·유상증자·전환사채·중대한 소송/규제 소식이 없다.
5. **하락 추세 완화**: 아래 [추세 판정](#추세-판정)의 두 조건을 모두 충족한다.

다음은 제외한다.

- 최근 7일 부모 메시지에 이미 나온 종목
- 최근 7일 안에 하루 상승률이 한국 15% 이상·미국 20% 이상이었던 종목

후보가 여러 개면 **신호 개수가 많은 종목 → 동종 대비 PER 할인이 큰 종목** 순으로 고른다.

## 후보 탐색

신호 6종마다 시장별 검색어로 WebSearch를 **최소 1회씩** 실행한다. 검색어 끝에 브리핑 날짜의 연·월(예: `2026년 11월` / `November 2026`)을 붙이고, 수집 기간이 두 달에 걸치면 두 달 모두 검색한다.

| 신호 | 🇰🇷 한국 검색어 | 🇺🇸 미국 검색어 |
|---|---|---|
| 실적 컨센서스 상회 | `어닝 서프라이즈 컨센서스 상회` | `earnings beat estimates` |
| 가이던스 상향 | `가이던스 상향 실적 전망 상향 발표` | `raises full-year guidance` |
| 목표주가·투자의견 상향 | `목표주가 상향 투자의견 상향` | `price target raised upgrade` |
| 수주·공급계약 | `단일판매 공급계약 체결 공시` | `wins contract awarded order` |
| 자사주 매입·소각 | `자사주 취득 소각 결정 공시` | `announces share repurchase program` |
| 배당 확대 | `배당 확대 주당배당금 상향 결정` | `raises quarterly dividend increase` |

- 결과 중 사건일·발행일이 수집 기간 안에 있는 종목만 후보로 모은다.
- 신호 하나에서 후보가 나오지 않으면 표현을 바꿔 1회 더 검색할 수 있다.
- 같은 종목이 여러 검색에서 나오면 하나로 합치고 신호 종류별로 한 번만 센다.
- 후보마다 종목코드(한국 6자리)·티커(미국)와 상장 시장을 확인한다. 확인하지 못하면 제외한다.

## 저평가 조회

EPS와 동종 PER은 **한국은 네이버 금융, 미국은 Yahoo Finance** 제공값만 쓴다. 아래 코드를 그대로 쓰고, 기사 속 PER·EPS로 대신하지 않는다.

| 시장 | 최근 4분기 EPS | 동종업계 |
|---|---|---|
| 🇰🇷 한국 | 네이버 종목 통합정보의 `eps` | 네이버 `동일업종 비교` 종목들의 PER |
| 🇺🇸 미국 | Yahoo `trailingEps` | Yahoo 같은 업종(industry) 상위 종목 최대 8개의 `trailingPE` |

- 동종 PER은 양수 값만 쓰고, 3개 이상일 때만 중앙값을 낸다. 3개 미만이면 저평가를 확인할 수 없으므로 제외한다.
- EPS가 0 이하이거나 없으면 PER을 낼 수 없으므로 제외한다.
- 동종 종목을 빼거나 더해 중앙값을 바꾸지 않는다.
- 결과의 `eps`·`peer_per_median`을 [매매가 계산](#매매가-계산)의 `levels()`에 그대로 넣는다.

```python
import json, statistics, urllib.request
from decimal import Decimal as D, InvalidOperation

def num(value):
    try:
        d = D(str(value).replace(",", "").replace("배", "").replace("원", "").strip())
    except InvalidOperation:
        return None
    return d if d.is_finite() and d > 0 else None

def naver(code):
    req = urllib.request.Request(f"https://m.stock.naver.com/api/stock/{code}/integration",
                                 headers={"User-Agent": "Mozilla/5.0"})
    with urllib.request.urlopen(req, timeout=10) as r:
        data = json.load(r)
    return data, {i.get("code"): i.get("value") for i in data.get("totalInfos", [])}

def valuation(market, code, close):
    peers = {}
    if market == "KR":
        data, info = naver(code)
        eps = num(info.get("eps"))
        for p in data.get("industryCompareInfo") or []:
            if p.get("itemCode") and p["itemCode"] != code:
                peers[p.get("stockName")] = num(naver(p["itemCode"])[1].get("per"))
        source = f"https://finance.naver.com/item/main.naver?code={code}"
    else:
        import yfinance as yf
        symbol = code.replace(".", "-")
        info = yf.Ticker(symbol).get_info()
        eps = num(info.get("trailingEps"))
        for s in [s for s in yf.Industry(info["industryKey"]).top_companies.index if s != symbol][:8]:
            peers[s] = num(yf.Ticker(s).get_info().get("trailingPE"))
        source = f"https://finance.yahoo.com/quote/{symbol}/key-statistics"
    values = [v for v in peers.values() if v]
    per = D(str(close)) / eps if eps else None
    median = statistics.median(values) if len(values) >= 3 else None
    return {"eps": eps, "per": per, "peer_per_median": median, "peers": peers, "source": source,
            "passed": per is not None and median is not None and per < median}
```

## 추세 판정

Yahoo Finance 수정종가(`Adj Close`, 분할·배당 반영)의 직전 거래일까지 연속 80거래일로 두 조건을 확인한다.

1. 직전 수정종가 ≥ 최근 60거래일 수정종가 평균
2. 최근 60거래일 평균 ≥ 20거래일 전 시점의 60거래일 평균

- 아래 코드를 그대로 쓴다. 한국은 `종목코드.KS`, 없으면 `종목코드.KQ`로 조회한다. 미국 티커의 `.`은 `-`로 바꾼다(예: `BRK-B`).
- 데이터가 80거래일보다 적거나 마지막 날짜가 직전 거래일이 아니면 추세를 확인할 수 없으므로 제외한다.
- 기사 속 주가·`바닥` 같은 표현으로 대신하지 않는다. 추세 통과는 향후 상승을 뜻하지 않는다.

```python
from datetime import timedelta
from decimal import Decimal as D

def trend(symbol, brief_date, last_session):
    import yfinance as yf
    h = yf.Ticker(symbol).history(start=(brief_date - timedelta(days=400)).isoformat(),
                                  end=brief_date.isoformat(), auto_adjust=False)
    rows = [(ts.date(), D(str(v))) for ts, v in h["Adj Close"].dropna().items() if ts.date() < brief_date][-80:]
    if len(rows) < 80 or rows[-1][0] != last_session:
        raise ValueError(f"추세 데이터 부족 또는 기준일 불일치: {len(rows)}개, 마지막 {rows[-1][0] if rows else None}")
    closes = [v for _, v in rows]
    ma60, ma60_prev = sum(closes[-60:]) / 60, sum(closes[:60]) / 60
    return {"date": rows[-1][0], "close": closes[-1], "ma60": ma60, "ma60_prev": ma60_prev,
            "eligible": closes[-1] >= ma60 and ma60 >= ma60_prev}
```

`last_session`은 종가에 쓴 직전 거래일이다. 필요하면 `python3 -m pip install -q yfinance`로 설치한다.

## 매매가 계산

선정 종목마다 아래 순서로 계산한다. 계산은 Python `Decimal`로 하고 아래 코드를 그대로 쓴다.

| 항목 | 계산 |
|---|---|
| 기준가치 | 최근 4분기 실제 EPS × 동종 PER 중앙값([저평가 조회](#저평가-조회) 결과). 목표주가가 확인됐고 더 낮으면 목표주가 |
| 매수상한 | 기준가치 × 0.80 (안전마진 20%) |
| 진입가 | 매수상한과 직전 종가 중 낮은 값 |
| 손절가 | 진입가 × 0.92 (−8%) |
| 수익실현가 | 진입가 × 1.16 (+16%). 기준가치보다 높으면 기준가치 |

- 매수상한·진입가·수익실현가는 호가 단위로 내림, 손절가는 올림한다. 한국은 KRX 호가 단위, 미국은 1센트다.
- 직전 종가가 매수상한 이하면 `가격조건 충족`, 초과면 `진입대기`다.
- 가격은 브리핑 날짜 + 28일까지 유효하다.
- EPS·동종 PER이 확인되지 않아 계산할 수 없으면 그 종목은 선정하지 않는다.
- 예상 EPS, 분기 EPS 연율화, 목표주가 역산으로 기준가치를 만들지 않는다.

```python
from datetime import date, timedelta
from decimal import Decimal as D, ROUND_CEILING, ROUND_FLOOR

def tick(price, market):
    if market == "US":
        return D("0.01")
    for limit, t in [(2000, 1), (5000, 5), (20000, 10), (50000, 50), (200000, 100), (500000, 500)]:
        if price < limit:
            return D(t)
    return D(1000)

def round_price(value, market, rounding):
    t = tick(value, market)
    return (value / t).to_integral_value(rounding=rounding) * t

def levels(close, eps, peer_per, target, market, brief_date):
    close, eps, peer_per = D(str(close)), D(str(eps)), D(str(peer_per))
    if eps <= 0 or peer_per <= 0 or close <= 0:
        raise ValueError("EPS·동종 PER·종가는 양수여야 함")
    value = eps * peer_per
    if target is not None:
        value = min(value, D(str(target)))
    ceiling = round_price(value * D("0.80"), market, ROUND_FLOOR)
    entry = round_price(min(close, ceiling), market, ROUND_FLOOR)
    stop = round_price(entry * D("0.92"), market, ROUND_CEILING)
    take_profit = round_price(min(value, entry * D("1.16")), market, ROUND_FLOOR)
    if not stop < entry < take_profit:
        raise ValueError("손절 < 진입 < 수익실현 조건 미충족")
    return {"value": value, "ceiling": ceiling, "entry": entry, "stop": stop,
            "take_profit": take_profit,
            "status": "가격조건 충족" if close <= ceiling else "진입대기",
            "until": brief_date + timedelta(days=28)}
```

매매가는 EPS와 동종 PER에 기초한 상대가치 참고값이며 수익이나 손실 한도를 보장하지 않는다. 목표주가는 보통 6~12개월 기준이므로 수익실현가로 쓰지 않는다.

## 조사 방법

- 뉴스·공시는 WebSearch로 찾고, 제목·URL·요약에서 확인한 사실만 쓴다.
- 링크는 확인한 원문 URL만 쓴다. 추적 파라미터(`utm_*`)와 Google News 리다이렉트는 쓰지 않는다.
- 수치·날짜·회사명은 확인한 것만 쓰고 추정하지 않는다.
- 주가·실적 예측이나 매매 권유 표현은 쓰지 않는다.

## 실행 순서

1. 브리핑 날짜와 기준 시각을 정한다.
2. Slack `#daily-stock-brief`(`C0CDFHG8PL7`)의 최근 7일 메시지를 읽는다.
   - 브리핑 날짜의 부모 메시지가 이미 있으면 **게시하지 않고 종료**한다.
   - 최근 7일 부모 메시지에 나온 종목을 중복 제외 목록으로 둔다.
3. [후보 탐색](#후보-탐색)의 검색어로 시장별 후보를 모은다.
4. 선정 기준을 확인하고 시장별로 1개를 고른 뒤 매매가를 계산한다.
5. 아래 형식으로 부모 메시지를 게시하고, 그 스레드에 한국 → 미국 순으로 답글을 단다.

## Slack 형식

모든 메시지는 한국어로 쓰고 메시지마다 3,000자 이하로 한다.

### 부모 메시지

```
📈 **Daily Stock Brief** | 2026-11-17 (화)
_수집 기간: 11/10 07:00 ~ 11/17 07:00 KST_

**🇰🇷 한국**
• 회사명 — 선정 이유 한 줄 · 진입대기

**🇺🇸 미국**
• 회사명 — 선정 이유 한 줄 · 가격조건 충족

🧵 자세한 내용은 스레드에서 확인
```

선정 종목이 없는 시장은 `_이번 조사에서 선정한 종목 없음_`으로 쓴다.

### 시장 답글

```
🇰🇷 **한국 종목**
**회사명 (005930)**
_무엇을 하는 회사인지 한 줄_

💵 **진입 64,000원 이하 (진입대기)** · 손절 58,900원 (−8%) · 수익실현 74,200원 (+16%)
_11/16 종가 71,200원 기준 · 12/15까지 유효_

• **신호** 최근 무슨 일이 있었나 ([매체명](원문링크), 11/14)
• **저평가·실적** PER 7.2배 (동종 중앙값 10.1배) · 최근 분기 영업이익 1,234억 원
• **리스크** 확인할 약점이나 반론

📊 추세: 수정종가 71,200원 ≥ 60일 평균 68,450원 (20거래일 전 평균 66,900원 이상) · Yahoo Finance, 11/16 기준
📐 매수상한 64,000원 (최근 4분기 EPS 8,000원 × 동종 PER 중앙값 10배 = 80,000원 · 안전마진 20%)
🎯 목표주가 90,000원 (A증권, 11/14) 또는 목표주가 미확인
```

`💵`·`📐` 줄은 `levels()` 결과를 그대로 쓴다. 목표주가가 기준가치보다 낮아 기준가치로 쓰였으면 `📐` 줄에 그 사실을 밝힌다.

선정 종목이 없는 시장도 답글을 달고, 선정하지 못한 이유를 한두 줄로 쓴다.
