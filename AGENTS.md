# Daily Stock Brief

매일 오전 7시(KST)에 Claude Code 루틴이 읽는 유일한 실행 지시서다.
한국·미국 상장 기업을 각각 최대 2개 골라 Slack `#daily-stock-brief`(ID `C0CDFHG8PL7`)에 부모 메시지와 스레드로 바로 게시한다. 사람의 검토 없이 실행하며 투자 권유가 아니다.
기업은 기사로 고르고, 매매가는 직전 거래일 종가에 정해진 공식을 적용한 참고값이다. 공시 원문·재무제표·밸류에이션 심층 분석은 하지 않는다.

> `CLAUDE.md`와 `README.md`는 이 파일의 심볼릭 링크다. 지침은 `AGENTS.md`만 수정한다.

## 독자와 범위

독자는 티맵모빌리티의 Data Engineer다. 원문 언어와 관계없이 한국어로 쓰며, 부모 메시지는 1분, 기업 스레드는 합쳐서 3분 안에 읽을 수 있게 한다.

| 섹션 | 대상 | 최대 개수 |
|---|---|---|
| 🇰🇷 한국 | 유가증권·코스닥 상장 기업 | 2개 |
| 🇺🇸 미국 | NYSE·Nasdaq 상장 기업 | 2개 |

양국에 상장된 기업은 주 상장 거래소로 분류한다. 조건에 맞는 기업이 부족하면 있는 만큼만 쓴다. 기업이 없는 섹션은 부모·답글 모두 헤더 아래에 `_해당 기간 새 기업 없음_`을 쓴다.

### 날짜 기준

`TZ=Asia/Seoul date`로 브리핑 날짜를 확인한다. 실제 실행 시각과 무관하게 **당일 07:00 KST**를 기준으로 다음 기간을 적용한다. 주말·휴장일에도 늘리지 않는다.

| 용도 | 기간·조건 |
|---|---|
| 신규 긍정 신호 | 직전 72시간 안에 기사 발행과 사건 발생이 모두 확인돼야 함 |
| 목표주가 보충·위험 점검 | 직전 30일 안에 발행된 기사 |
| 기업 중복 확인 | 오늘을 제외한 최근 7일의 KST 날짜 |

- **발행일**은 URL의 날짜 또는 검색 결과에 직접 나온 발행일로 확인한다. 기사 번호·내용·앞뒤 기사로 추정하지 않는다.
- **사건일**은 검색 결과의 실적·리포트 발표일, 계약·공시일 등으로 확인한다. `이날 발표`처럼 발행일과의 관계가 명시돼도 된다. 발행일만으로 추정하거나 과거 사건의 재소개를 새 신호로 쓰지 않는다.
- 시각·시간대가 있으면 KST로 환산한다. 기간 경계에서 날짜만으로 포함 여부를 판단할 수 없으면 해당 신호를 인정하지 않는다.
- 30일 기사에서 찾은 목표가 상향도 기사·리포트 발표일이 모두 72시간 안일 때만 신호 점수를 준다.

## 소스와 조회 원칙

- 기사·공시는 직접 열지 않고 **WebSearch의 제목·URL·요약**으로 확인한다. DART·EDGAR 공시는 보도 기사로 확인한다.
- 종가·일간 등락률·시가총액은 아래 **시세 조회** 스크립트를 쓴다. 그 밖의 시장 수치와 PER·PBR 등은 기사에 나온 것만 쓴다.
- 링크는 검색 결과에서 확인한 원문 URL만 쓰고 `utm_*` 등 추적 파라미터를 제거한다. Google News 리다이렉트 URL은 쓰지 않는다.
- 기업별 긍정 신호 대표 링크는 가장 신뢰도 높은 원문 하나를 쓴다. 별도 위험 점검 기사 내용을 본문에 인용하면 발행일과 근거 링크도 붙인다.
- `#daily-news-brief`는 읽지 않으며, 그 채널과의 기업 중복은 제한하지 않는다.

루틴 환경에서 WebFetch 접근이 제한되므로 검색 결과를 사용한다. 다음 매체는 **도메인 필터에 넣지 않는다**. 필터 없는 검색 결과에 나오면 링크로 사용할 수 있다:
Reuters, AP, BBC, NYT, The Guardian, Politico(EU), The Verge, Ars Technica, USA Today, The Independent, El País, 연합뉴스(`yna.co.kr`).

| 검색 후보 | 매체 |
|---|---|
| 한국 | 머니투데이(`mt.co.kr`), 파이낸셜뉴스(`fnnews.com`), 아주경제(`ajunews.com`), 이투데이(`etoday.co.kr`), 서울신문(`seoul.co.kr`), 한국일보(`hankookilbo.com`), 경향신문(`khan.co.kr`), 세계일보(`segye.com`), 노컷뉴스(`nocutnews.co.kr`), 뉴시스(`newsis.com`), 헤럴드경제(`heraldcorp.com`), SBS(`news.sbs.co.kr`) |
| 한국 증권 | 한국경제(`hankyung.com`), 매일경제(`mk.co.kr`), 서울경제(`sedaily.com`), 이데일리(`edaily.co.kr`) |
| 미국 | CNBC, Bloomberg, Yahoo News, CNN, NBC News, CBS News, ABC News, NPR |
| 미국 증권 | Yahoo Finance, MarketWatch, Barron's, Investor's Business Daily |

검색 결과가 부족하면 다른 매체를 사용해도 된다. 다음뉴스(`v.daum.net`)는 원문 URL을 찾지 못했을 때만 쓴다.

## 기업 선정

### 긍정 신호와 점수

다음 중 하나 이상을 충족해야 한다. 검색 요약에 근거가 부족하면 추가 검색하고, 끝내 확인하지 못한 항목은 0점으로 처리한다.

| 신호 | 점수 | 인정 조건 |
|---|---|---|
| 실적 컨센서스 상회 | 2 | 매출·영업이익·순이익·EPS 중 지표를 밝히고, 같은 기간·통화·회계 기준의 실제치와 컨센서스를 확인. 전년 대비 증가만으로 인정하지 않으며 GAAP·조정 수치를 섞지 않음 |
| 가이던스 상향 | 2 | 회사가 상향을 발표하고 동일 지표·기간의 기존 값과 새 값을 확인. 범위는 양쪽 모두 확인하며 상단 하나만으로 추정하지 않음. 최초 제시·유지·애널리스트 전망 상향은 제외 |
| 목표주가·투자의견 상향 | 증권사 1곳 1점, 2곳 이상 2점 | 증권사·리포트 발표일·상향 사실 확인. 유지·최초 커버리지·과거 상향 재소개는 제외 |
| 수주·공급계약 공시 | 1 | 계약 대상과 체결·수주 공시 확인. MOU·협상·기대는 제외하며 상대방·금액·기간은 보도된 경우만 사용 |
| 자사주 매입·소각 | 1 | 신규 결정 또는 실제 이행 공시와 금액 또는 주식 수 확인. 결정·이행을 구분하며 계획 재소개·의향 표명은 제외 |
| 배당 확대 | 1 | 같은 기준의 기존·새 주당 배당금과 확대 결정 공시 확인. 유지·검토·주가 하락에 따른 배당수익률 상승은 제외 |

각 항목은 한 번만 센다. 같은 실적의 여러 기사나 같은 자사주 계획의 결정·이행을 중복 가산하지 않는다.

### 제외 조건과 위험 점검

- 주가 상승만으로 긍정 신호를 인정하지 않는다.
- 시세 조회의 `daily_change_pct`에 한국 하루 15% 이상·미국 20% 이상 상승이 있거나, 72시간 기사에서 상한가·같은 수준의 급등이 확인되면 제외한다.
- 아래 **위험 검색을 마친 후보만** 확정한다. 대체 후보도 동일하다.

후보마다 회사명(가능하면 종목코드·티커 포함)과 30일 기간을 지정해 두 묶음을 각각 WebSearch한다. 한국은 한국어, 미국은 영어로 검색한다.

| 묶음 | 한국어 | 영어 |
|---|---|---|
| 실적·전망 | `실적 하회 OR 컨센서스 하회 OR 가이던스 하향` | `earnings miss OR revenue miss OR guidance cut OR lowered outlook` |
| 희석·규제·소송 | `유상증자 OR 전환사채 OR 규제 OR 소송` | `share offering OR convertible notes OR regulatory action OR lawsuit` |

- **제외**: 30일 기사에서 해당 기업의 실적 하회·가이던스 하향·희석 공시·규제 조치·소송을 확인한 경우. 업종 제도 설명이나 일반적인 가능성만으로 제외하지 않는다.
- **위험 점검 미완료**: 관련 악재의 발행일·대상·사건을 추가 검색해도 확인하지 못하거나 필수 검색이 실패한 경우. 이 후보는 확정하지 않고 다른 후보를 계속 점검한다.
- **해당 검색에서 제외 사유 미발견**: 두 검색을 정상 완료했으나 제외 근거를 찾지 못한 경우. 이를 `위험 없음`·`안전`으로 표현하지 않는다.

검색어 두 묶음·기간·판정·근거는 최종 실행 기록에 남긴다. 발견한 기사에는 URL·발행일·사건 요약을 기록한다.

### 상승여력과 선정 순서

상승여력은 목표주가 ÷ 직전 거래일 종가 − 1이다. **가격 계산**의 `levels()`가 반환한 `upside_raw`로 판단한다.

- **15% 미만이면 제외**한다. 목표가 또는 직전 거래일 종가가 없으면 `미확인`이다.
- 가점은 15% 이상이면 1점, 미확인이면 0점이다. 총점은 신호 점수 + 가점으로 기록한다.
- 시장별로 상승여력 확인 후보를 먼저 고른다. 그 안에서 **신호 점수 → 이미 고른 기업과 다른 업종 → 주요 매체 수 → 상승여력 원값** 순으로 비교한다. 첫 기업을 고를 때는 업종 비교를 생략한다.
- 확인 후보가 2개 미만이면 미확인 후보로 빈자리만 채운다. 신호 점수·업종·매체 수 순서를 적용한다.
- 시가총액은 규모 참고용이다. 점수·동점 처리·제외 조건이나 저평가·안정성 판단에 쓰지 않는다.

## 중복 방지와 재실행

채널의 `📈 Daily Stock Brief` 제목과 날짜를 기준으로 부모 메시지를 식별한다. 도구가 돌려주는 굵게 표기 유무는 무시한다.

- 최근 7일 부모 메시지 전부에서 기업을 확인해 제외한다. 이때 스레드와 사람이 쓴 다른 메시지는 읽지 않는다.
- 오늘 안에서도 같은 기업을 두 번 쓰지 않는다.
- **오늘 부모 메시지가 있으면 새로 만들지 않고 해당 스레드만 읽는다.** 한국·미국 섹션 답글이 모두 있으면 아무것도 게시하지 않고 종료한다. 분할 게시였다면 이어지는 답글까지 완성됐는지 확인한다.
- 빠진 섹션은 부모에 적힌 기업·순서로만 작성하고 위험 점검을 다시 한다. 분할 답글 일부가 이미 있으면 게시된 항목은 건너뛰고 누락 항목만 이어 쓴다. 제외·미완료 기업은 같은 위치에 이름과 보류 사유만 쓰며 대체 기업·가격 블록은 넣지 않는다. 이 보류 표시는 일반 기업 항목·빈 섹션 형식의 예외다.
- 재실행 가격은 부모 작성 당시의 종가·목표가를 찾아 상승여력을 맞춘다. 찾지 못하면 새 값을 쓰고 💵 제목 끝에 `(부모 메시지 작성 후 갱신)`을 붙인다. 기간은 부모 날짜 + 28일이다.

## Slack 작성

부모 1개와 한국·미국·실행 정보 답글 순으로 게시한다. 일반 실행에서 기업이 없는 시장도 빈 섹션 답글을 쓴다.

- 메시지는 URL 포함 **3,000자 이하**인지 코드로 센다. 문장·URL을 줄여도 넘으면 완전한 기업 항목 사이에서만 나누고 제목에 이어지는 답글임을 표시한다.
- 표준 Markdown을 쓴다: 굵게 `**텍스트**`, 기울임 `_텍스트_`, 링크 `[텍스트](URL)`.
- 회사명은 한국은 한글, 미국은 영문으로 쓰며 부모·스레드에서 표기·순서를 맞춘다.
- 사실·수치·일정·고유명사는 기사나 시세 조회로 확인한 것만 쓴다. 주가·등락률에는 종가·장중·시간외 여부와 기준일을 붙이고 서로 다른 기준을 섞지 않는다.
- 매매 권유와 주가·실적 예측은 쓰지 않는다. 전망은 `회사는 …로 제시했다`, `A증권은 …로 올렸다`처럼 주체를 밝힌 인용만 허용한다. 가격 움직임만으로 펀더멘털을 판단하지 않는다.

### 부모 메시지

```
📈 **Daily Stock Brief** | 2026-11-02 (월)
_수집 기간: 10/30 07:00 ~ 11/02 07:00 KST_

**🇰🇷 한국**
• 회사명 — 선정 이유 한 줄 · 상승여력 26%
• 회사명 — 선정 이유 한 줄 · 상승여력 18%

**🇺🇸 미국**
• 회사명 — 선정 이유 한 줄 · 상승여력 21%
• 회사명 — 선정 이유 한 줄 · 상승여력 미확인

🧵 투자 포인트와 매매가는 스레드에서 확인 · 매매가는 직전 거래일 종가 기준 공식값 · 투자 권유 아님
```

- 날짜는 KST의 `YYYY-MM-DD (한글 요일 한 글자)`로 쓴다.
- 선정 이유는 스레드 첫 포인트를 한 줄로 줄이며, 여러 신호면 함께 쓴다.
- 상승여력은 목표주가 줄의 `upside_pct`와 일치시킨다. 매매가 미산출·목표가 미확인은 `상승여력 미확인`으로 쓰며 수익실현율과 혼동하지 않는다.

### 기업 답글

번호는 시장마다 1부터 시작한다. 다음은 형식 예시다.

```
🇰🇷 **한국 기업**
**1. 회사명 (005930)**
_회사 설명 1줄 · 시가총액 12.3조 원 (네이버, 11/2 07:02 KST 조회)_
• 최근 무슨 일이 있었나
• 왜 중요한가
• 리스크 또는 다음에 확인할 것
💵 가격 (10/30 종가, 네이버 · PER 12.3배 A증권 10/30 · 기간 11/30까지)
• 목표주가 90,000원 (A증권, 10/30 상향 · 상승여력 26%)
• 수익실현 82,500원 (+16%)
• 진입 71,200원 이하
• 손절 65,600원 (−8%)
🔗 [매체명](https://원문링크)

**2. ...**
```

- **제목**: 한국은 6자리 종목코드, 미국은 티커를 검색 결과로 확인해 괄호에 넣는다. 확인하지 못하면 괄호를 생략하고 시가총액은 미확인으로 쓴다.
- **설명 1줄**: 무엇을 만들거나 파는 회사인지 한 문장 뒤에 `market_cap.display`를 붙인다. 시가총액은 한국 원화·미국 달러이며 조·억·만 단위로 소수 첫째 자리까지 반올림하고 `.0`을 생략한다. 출처·KST 조회 시각은 종가 기준일이나 데이터 갱신 시각으로 해석하지 않는다. 실패 시 기사·주식 수 계산으로 보충하지 않는다.
- **투자 포인트는 정확히 3줄**, 각 한 문장으로 쓴다.
  1. **사건**: 인정된 신호의 사건일·수치. 실적은 지표·기간·실제치·컨센서스, 가이던스는 기존·새 값, 자사주는 결정·이행 구분을 밝힌다.
  2. **의미**: 매출·이익·사업에 미치는 의미를 기사 근거로 설명한다. 상회한 지표 하나를 전체 실적 상회로 확대하지 않는다.
  3. **리스크·다음 확인**: 기사에 나온 약점·반론·일회성 요인을 우선한다. 없으면 무엇이 확인될 때 판단이 달라지는지 조건으로 쓴다. 확인 일정은 기사에 있는 것만 쓴다. 제외 사유가 있는 기업을 이 줄로 설명해 통과시키지 않는다.

### 가격 블록

분석 기업마다 설명·포인트 아래에 작성한다. 차트·지지선·저항선으로 공식을 바꾸지 않는다.

제목은 `💵 가격 (기준일 종가, 출처 · 밸류에이션 · 기간 M/D까지)`다.
기준일·출처는 진입가에 쓴 값의 `date`·`source`이며, 밸류에이션은 기사에 PER·PBR이 있을 때만 출처·기준일과 함께 쓴다. 기간은 브리핑 날짜 + 28일이며, 그때까지 손절·수익실현에 닿지 않으면 다음 거래일에 정리한다는 기준이다.

가격 줄은 다음 순서를 유지한다. 한국은 원, 미국은 달러(`$182.40`)로 쓴다.

| 줄 | 값·표기 |
|---|---|
| 목표주가 | 최근 증권사·애널리스트 리포트를 보도한 기사 하나의 목표가, 증권사·리포트 날짜·상향/하향 여부·상승여력 `upside_pct` |
| 수익실현 | `min(목표주가, 진입가 × 1.16)`. 목표가 미확인이면 진입가 × 1.16. 괄호는 `(+N%)`로 `take_profit_pct`만 표시 |
| 진입 | 직전 거래일 종가(`close`) 뒤에 `이하`를 붙임. 현재가 줄을 따로 쓰지 않음 |
| 손절 | 진입가 × 0.92, 괄호는 `(−8%)` |

- 목표가는 보통 6~12개월 기준이므로 4주 수익실현가로 그대로 쓰지 않는다. 선정 기사에 없으면 `회사명 목표주가` / `티커 price target`으로 추가 검색한다. 30일 안의 기사로 확인하지 못하면 `• 목표주가 미확인`으로 쓰고, 컨센서스 평균·오래된 값으로 채우지 않는다.
- 한국은 KRX 호가 단위, 미국은 1센트 단위로 손절은 올림·수익실현은 내림한다. **반드시 아래 코드로 계산**하고, 상승여력 판정은 반올림 전 값으로 한다.

#### 종가 조회 실패 시

최상위 `error` 또는 다른 동일 시장 후보보다 오래된 `date`이면 기사 가격을 확인한다. 선정 기사에 없으면 `회사명 주가` / `티커 stock price`로 한 번 더 검색해 72시간 안의 기사에서 찾는다.

- **직전 거래일 종가 확인**: 그 값으로 계산하고 가격 제목의 출처를 매체명으로 쓴다.
- **확인 불가**: 제목은 `💵 가격 (매매가 미산출: 직전 거래일 종가 없음)`으로 쓰고 아래 두 줄만 둔다. 목표가에는 상승여력을 붙이지 않는다.
  - `• 목표주가 …` 또는 `• 목표주가 미확인`
  - `• 현재가 72,000원 (10/30 장중, 매체명)` 또는 `• 현재가 미확인 (시세 조회 실패, 수집 기간 기사에 없음)`

장중·오래된 종가·추정값으로 매매가를 계산하지 않으며, 조회 오류는 최종 기록에 남긴다.

### 가격 계산

```python
import math
from datetime import date, timedelta

def krx_tick(p):  # KRX 호가 단위 (유가증권·코스닥 공통)
    for limit, t in [(2000, 1), (5000, 5), (20000, 10), (50000, 50), (200000, 100), (500000, 500)]:
        if p < limit:
            return t
    return 1000

# price: 직전 거래일 종가, target: 목표주가(미확인이면 None), market: "KR" 또는 "US"
def levels(price, target, market, brief_date):
    # 목표주가는 12개월 기준이라 4주 기간에 맞게 +16%를 넘지 않게 한다
    stop, tp = price * 0.92, min(target or price * 1.16, price * 1.16)
    if market == "KR":
        stop = math.ceil(round(stop / krx_tick(stop), 6)) * krx_tick(stop)
        tp = math.floor(round(tp / krx_tick(tp), 6)) * krx_tick(tp)
    else:
        stop = math.ceil(round(stop * 100, 6)) / 100
        tp = math.floor(round(tp * 100, 6)) / 100
    upside = target / price - 1 if target else None
    return {"stop": stop, "take_profit": tp, "take_profit_pct": round((tp / price - 1) * 100),
            "upside_raw": upside,
            "upside_pct": round(upside * 100) if upside is not None else None,
            "until": (brief_date + timedelta(days=28)).strftime("%-m/%-d")}

print(levels(71200, 90000, "KR", date(2026, 11, 2)))
```

## 시세 조회

후보를 확정하기 전에 아래 명령에 `KR:종목코드` / `US:티커`를 한 번에 넘긴다. 종목마다 JSON 한 줄을 출력한다.

- 종가·등락률은 한국 네이버 우선, 실패 시 Yahoo Finance(`.KS` → `.KQ`), 미국은 Yahoo Finance다.
- 시가총액은 한국 네이버 시세 API, 미국 Yahoo Finance의 `marketCap` 제공값이다. 종가 조회와 독립 처리하며 네이버는 `yfinance` 설치 실패 시에도 사용할 수 있다.
- `date`·`close`는 브리핑 날짜 이전의 가장 최근 거래일 종가다. 오늘 행은 제외한다. `daily_change_pct`는 브리핑 날짜 − 3일 이후의 일간 등락률이다.
- `market_cap`은 원/달러 값(`value`), 통화(`currency`), 출처(`source`), KST 조회 시각(`retrieved_at`), 표시 문구(`display`)다. `market_cap.error`만 있으면 종가 조회 실패로 취급하지 않고 `시가총액 미확인`을 표시한다.

```bash
pip install -q yfinance 2>/dev/null
python3 - KR:005930 US:NVDA 2>/dev/null <<'EOF'
import ast, json, sys, urllib.request
from datetime import datetime, timedelta, timezone
from decimal import Decimal, ROUND_HALF_UP

KST = timezone(timedelta(hours=9))
today = datetime.now(KST).date()  # 브리핑 날짜 (KST)
window_start = today - timedelta(days=3)  # 수집 기간이 걸친 첫 날짜

def naver(code):
    url = (f"https://api.finance.naver.com/siseJson.naver?symbol={code}&requestType=1&timeframe=day"
           f"&startTime={today - timedelta(days=30):%Y%m%d}&endTime={today:%Y%m%d}")
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
    text = urllib.request.urlopen(req, timeout=10).read().decode("utf-8", "replace")
    return [(datetime.strptime(r[0], "%Y%m%d").date(), r[4]) for r in ast.literal_eval(text.strip())[1:] if r[4]]

def yahoo(symbol):
    import yfinance as yf
    h = yf.Ticker(symbol).history(period="1mo", auto_adjust=False)
    return [(ts.date(), round(float(c), 2)) for ts, c in h["Close"].dropna().items()]

def format_cap(value, currency):
    for scale, unit in [(10**12, "조"), (10**8, "억"), (10**4, "만"), (1, "")]:
        if value >= scale:
            amount = (value / scale).quantize(Decimal("0.1"), rounding=ROUND_HALF_UP)
            number = f"{amount:,.1f}".removesuffix(".0")
            return f"{number}{unit} {'원' if currency == 'KRW' else '달러'}"

def market_cap(market, code):
    source = "네이버" if market == "KR" else "Yahoo Finance"
    try:
        expected = "KRW" if market == "KR" else "USD"
        if market == "KR":
            url = f"https://polling.finance.naver.com/api/realtime/domestic/stock/{code}"
            req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
            with urllib.request.urlopen(req, timeout=10) as response:
                data = json.load(response)
            item = next(r for r in data["datas"] if r.get("itemCode") == code)
            raw = item.get("marketValueFullRaw")
            currency = (item.get("currencyType") or {}).get("code")
        else:
            import yfinance as yf
            symbol = code.replace(".", "-")
            info = yf.Ticker(symbol).get_info()
            if info.get("symbol") != symbol:
                raise ValueError("시가총액 종목 불일치")
            raw, currency = info.get("marketCap"), info.get("currency")
        if currency != expected:
            raise ValueError(f"시가총액 통화 불일치: {currency!r} (기대: {expected})")
        if raw is None or isinstance(raw, bool):
            raise ValueError("시가총액 값 없음")
        value = Decimal(str(raw))
        if not value.is_finite() or value < 1:
            raise ValueError("시가총액 값이 유효하지 않음")
        checked = datetime.now(KST)
        return {"value": int(value), "currency": currency, "source": source,
                "retrieved_at": checked.isoformat(timespec="seconds"),
                "display": f"시가총액 {format_cap(value, currency)} ({source}, {checked:%-m/%-d %H:%M} KST 조회)"}
    except Exception as e:
        return {"display": "시가총액 미확인", "source": source,
                "error": f"{type(e).__name__}: {e}"[:200]}

def quote(market, code):
    if market == "KR":
        sources = [("네이버", lambda: naver(code)), ("Yahoo Finance", lambda: yahoo(f"{code}.KS")),
                   ("Yahoo Finance", lambda: yahoo(f"{code}.KQ"))]
    else:
        sources = [("Yahoo Finance", lambda: yahoo(code.replace(".", "-")))]
    errors = []
    for name, fetch in sources:
        try:
            rows = [r for r in fetch() if r[0] < today]  # 오늘 날짜 행은 장중 가격일 수 있으므로 뺀다
        except Exception as e:
            errors.append(f"{name}: {type(e).__name__}: {e}"[:200])
            continue
        if len(rows) < 2:
            errors.append(f"{name}: 데이터 없음")
            continue
        changes = {f"{d:%-m/%-d}": round((c / p - 1) * 100, 2)
                   for (d, c), (_, p) in zip(rows[1:], rows) if d >= window_start}
        return {"market": market, "code": code, "source": name, "date": f"{rows[-1][0]:%-m/%-d}",
                "close": rows[-1][1], "daily_change_pct": changes}
    return {"market": market, "code": code, "error": errors}

for arg in sys.argv[1:]:
    market, code = arg.split(":", 1)
    result = quote(market, code)
    result["market_cap"] = market_cap(market, code)
    print(json.dumps(result, ensure_ascii=False))
EOF
```

## 실행 정보

한국·미국 섹션 답글 중 하나라도 **게시를 시도한 실행**에만 집계·게시한다. 해당 실행에서 필요한 섹션의 시도를 마친 뒤 실행하며, 답글이 3번 실패했어도 진행한다. 오늘 답글이 이미 완성돼 아무것도 시도하지 않은 재실행에서는 게시하지 않는다.

다음 명령은 Claude Code의 메인 JSONL과 그 세션의 서브에이전트 기록을 합산한다. 출력의 `file`이 **이번 루틴의 기록인지 확인한 뒤** 사용한다. 최신 파일이라는 이유만으로 현재 세션이라고 단정하지 않으며, 확인할 수 없으면 `집계 불가 (현재 세션 기록 미확인)`으로 쓴다.

```bash
python3 - <<'EOF'
import glob, json, os
from datetime import datetime, timezone, timedelta

mains = set()
for root in {os.path.expanduser("~"), "/root", "/home/user"}:
    mains.update(glob.glob(f"{root}/.claude/projects/*/*.jsonl"))
if not mains:
    print(json.dumps({"error": "기록 파일 없음"}))
    raise SystemExit

main = max(mains, key=os.path.getmtime)
subs = sorted(glob.glob(f"{main[:-len('.jsonl')]}/subagents/*.jsonl"))

def read(path):
    usage, first_ts, models = {}, None, set()
    with open(path, encoding="utf-8") as f:
        for n, line in enumerate(f):
            try:
                e = json.loads(line)
            except ValueError:
                continue
            first_ts = first_ts or e.get("timestamp")
            m = e.get("message")
            if e.get("type") == "assistant" and isinstance(m, dict) and m.get("usage"):
                # 한 응답이 여러 줄로 기록되면 같은 message.id가 반복되므로 마지막 값만 쓴다
                usage[m.get("id") or n] = m["usage"]
                if m.get("model"):
                    models.add(m["model"])
    return usage, first_ts, models

usage, first_ts, models = read(main)
main_responses = len(usage)
for p in subs:
    u, _, ms = read(p)
    usage.update({(p, k): v for k, v in u.items()})
    models |= ms

keys = ["input_tokens", "output_tokens", "cache_creation_input_tokens", "cache_read_input_tokens"]
result = {k: sum(u.get(k) or 0 for u in usage.values()) for k in keys}
result.update(responses=len(usage), main_responses=main_responses, subagents=len(subs),
              models=sorted(models), file=main)
if first_ts:
    start = datetime.fromisoformat(first_ts.replace("Z", "+00:00"))
    result["start_kst"] = start.astimezone(timezone(timedelta(hours=9))).strftime("%H:%M")
    result["elapsed_sec"] = int((datetime.now(timezone.utc) - start).total_seconds())
print(json.dumps(result, ensure_ascii=False))
EOF
```

숫자는 천 단위 쉼표를 넣고 `elapsed_sec`을 분·초로 변환한다. 서브 응답 수는 `responses - main_responses`다. 값은 추정하지 않으며, 집계 뒤 마지막 몇 턴은 포함되지 않는다. 명령 실패·오류 시 해당 줄에 `집계 불가 (오류 내용)`을 쓰고 나머지는 확인한 값만 쓴다.

```
⚙️ 실행 정보
• 토큰: 입력 12,345 · 출력 34,567 · 캐시 쓰기 123,456 · 캐시 읽기 1,234,567
• 응답: 62회 (메인 62 · 서브에이전트 0개 0)
• 소요 시간: 6분 37초 (시작 07:04 KST, 집계 시점까지)
• 모델: claude-opus-5-5
```

## 실행·게시 순서

1. **기준 설정**: 날짜 기준의 세 기간을 계산한다.
2. **채널 확인**: ID `C0CDFHG8PL7`로 읽고, 실패하면 `slack_search_channels`로 `daily-stock-brief`를 찾는다. 중복 목록을 만들고, 오늘 부모가 있으면 **중복 방지와 재실행** 규칙을 따른다.
3. **후보 조사**: 신규 신호·발행일·사건일·인정 근거를 확인하고 후보별 위험 점검을 수행한다.
4. **후보 확정**: 남은 후보의 시세·목표가를 조회해 `levels()`로 계산하고 제외 조건·선정 순서를 적용한다. 대체 후보도 같은 절차를 거친다.
5. **작성·검증**: 부모와 기업 답글을 작성하고 아래 게시 전 점검을 통과시킨다.
6. **직접 게시**: `slack_send_message`로 부모를 보내고 응답의 `ts`를 `thread_ts`로 써서 한국·미국 순으로 답글을 보낸다. 초안 도구는 쓰지 않는다. `reply_broadcast`·`unfurl_app_links`는 설정하지 않는다.
7. **마무리**: 실행 정보를 집계해 마지막 답글로 보내고, 아래 최종 결과를 남긴다.

### 게시 전 점검

게시한 메시지는 수정하지 않는다. 틀린 곳은 전송 전에 고친다.

- **적격성**: 날짜·중복·신호 인정·위험 점검·급등·상승여력 조건을 통과했으며 순위와 기록이 일치하는가.
- **내용**: 설명 1줄·포인트 3줄이 정해진 순서이고, 모든 사실·수치·전망 인용에 근거가 있으며 권유·예측 문장이 없는가.
- **가격**: 직전 거래일 종가·목표가 출처와 기준일, `levels()` 결과·호가 단위·기간이 맞는가. 미산출이면 예외 두 줄 형식인가.
- **일치**: 부모·답글의 회사명·순서·상승여력이 같은가. 재실행 보류·가격 갱신을 명시했는가.
- **표시**: 시가총액은 `display` 그대로이며 조회 시각을 종가 기준일로 쓰지 않았는가. 링크·Markdown이 유효하고 각 메시지 길이를 코드로 확인했는가.

### 실패·재시도

- 실패한 메시지만 재시도한다. 먼저 채널 또는 스레드를 다시 읽어 실제 게시 여부를 확인하고, 이미 있으면 다시 보내지 않는다.
- 채널을 찾지 못하거나 부모가 3번 실패하면 종료한다. 부모가 없으면 답글도 보내지 않는다.
- 답글 하나가 3번 실패하면 건너뛰고 나머지를 계속한다. 성공·실패 메시지, 시도 횟수·오류는 최종 기록에 남긴다.

## 최종 결과

루틴의 마지막 응답에만 남기며 Slack에는 올리지 않는다.

```
상태: 완료 / 일부 게시 / 게시 안 함 / 실패
부모 메시지: Slack 링크 또는 없음
답글: 한국 n개 기업(이름) · 미국 n개 기업(이름) · 실행 정보 게시 여부

생략·대체:
- 빈 섹션·미달·분할 게시·채널 검색 대체·재실행·보류·가격 갱신 사유
- 종가·시가총액 조회 실패 기업, 대체 출처·매매가 미산출 여부·오류

후보·선정 근거:
- 선정 기업과 탈락 상위 후보의 신호 점수·가점·총점·탈락 사유
- 인정 신호의 기사 발행일·사건일·URL과 실제치/컨센서스·기존/새 값 등
- 근거 부족으로 0점인 항목과 이유

위험 점검:
- 점검한 모든 후보의 검색어 두 묶음·기간·판정
- 제외·미완료 근거의 URL·발행일·사건 요약 또는 검색 오류

오류:
- 실패한 메시지(부모/한국/미국/실행 정보)·시도 횟수·오류, 없으면 없음
```
