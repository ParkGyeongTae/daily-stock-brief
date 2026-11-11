# Daily Stock Brief

매일 오전 7시(KST)에 Claude Code 루틴이 읽는 유일한 실행 지시서다.
한국·미국 상장 기업을 각각 최대 2개 골라 Slack `#daily-stock-brief`(ID `C0CDFHG8PL7`)에 부모 메시지와 스레드로 바로 게시한다. 사람의 검토 없이 실행하며 투자 권유가 아니다.
최근 호재가 있고, 기사로 정량적인 저평가 근거와 실적 지속성을 확인했으며, 시세로 하락 추세 완화 조건을 통과한 기업만 고른다. 진입가는 기사에서 확인한 주당 실적·자산과 비교배수로 산출한 매수상한에 제한한다. 공시 원문·재무제표를 직접 분석하거나 미래 실적·자산을 자체 추정하지 않는다. 저평가 근거와 진입가는 상대가치에 기초한 참고값이며 수익이나 가격 하한을 보장하지 않는다.

> `CLAUDE.md`와 `README.md`는 이 파일의 심볼릭 링크다. 지침은 `AGENTS.md`만 수정한다.

## 독자와 범위

독자는 티맵모빌리티의 Data Engineer다. 원문 언어와 관계없이 한국어로 쓰며, 부모 메시지는 1분, 기업 스레드는 합쳐서 3분 안에 읽을 수 있게 한다.

| 섹션 | 대상 | 최대 개수 |
|---|---|---|
| 🇰🇷 한국 | 유가증권·코스닥 상장 기업 | 2개 |
| 🇺🇸 미국 | NYSE·Nasdaq 상장 기업 | 2개 |

양국에 상장된 기업은 주 상장 거래소로 분류한다. 조건에 맞는 기업이 부족하면 있는 만큼만 쓴다. 미확인 후보로 빈자리를 채우지 않는다. 기업이 없는 섹션은 부모·답글 모두 헤더 아래에 `_선정 조건을 충족한 기업 없음_`을 쓴다.

### 날짜 기준

실행 시작 시 `TZ=Asia/Seoul date +%F`로 브리핑 날짜를 한 번 확인해 고정하고, 시세 조회와 가격 계산에도 같은 날짜를 전달한다. 실제 실행 시각과 무관하게 **브리핑 날짜 07:00 KST**를 기준으로 다음 기간을 적용한다. 주말·휴장일에도 늘리지 않는다.

| 용도 | 기간·조건 |
|---|---|
| 신규 긍정 신호 | 직전 72시간 안에 기사 발행과 사건 발생이 모두 확인돼야 함 |
| 저평가·실적 지속성 확인, 목표주가 보충·위험 점검 | 직전 30일 안에 발행된 기사 |
| 기업 중복 확인 | 오늘을 제외한 최근 7일의 KST 날짜 |

- **발행일**은 URL의 날짜 또는 검색 결과에 직접 나온 발행일로 확인한다. 기사 번호·내용·앞뒤 기사로 추정하지 않는다.
- **사건일**은 검색 결과의 실적·리포트 발표일, 계약·공시일 등으로 확인한다. `이날 발표`처럼 발행일과의 관계가 명시돼도 된다. 발행일만으로 추정하거나 과거 사건의 재소개를 새 신호로 쓰지 않는다.
- 시각·시간대가 있으면 KST로 환산한다. 기간 경계에서 날짜만으로 포함 여부를 판단할 수 없으면 해당 신호를 인정하지 않는다.
- 30일 기사에서 찾은 목표가 상향도 기사·리포트 발표일이 모두 72시간 안일 때만 신호 점수를 준다.
- 기사·리포트는 브리핑 날짜 07:00 KST까지 확인 가능한 자료만 사용한다. 종가·추세의 기준일은 직전 거래일로 고정하고, 시가총액은 별도로 표시한 조회 시각의 제공값을 쓴다. 실적 지속성에 쓰는 실제 실적은 이 기간 기사에 보도된 최신 발표 결산기 자료이며, 기사 발행일과 실적 대상 기간을 구분한다.

## 소스와 조회 원칙

- 기사·공시는 직접 열지 않고 **WebSearch의 제목·URL·요약**으로 확인한다. DART·EDGAR 공시는 보도 기사로 확인한다.
- 종가·일간 등락률·시가총액·추세 지표는 아래 **시세 조회** 스크립트를 쓴다. 그 밖의 시장 수치와 PER·PBR 등은 기사에 나온 것만 쓴다. 아래에 명시한 PER·PBR의 가격 보정·상대 할인율·가치기준 진입가 계산만 허용하며, 이익·자산·미래 매출을 자체 추정하지 않는다.
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

선정은 **긍정 신호 → 저평가 근거 → 실적 지속성 → 위험 점검 → 시세·추세 확인 → 가치기준 진입가 산출** 순으로 진행한다. 여섯 조건은 모두 필수다. 자사주 소각·배당 확대·목표가 상향은 조사 계기이며, 다른 조건을 대신하지 못한다. 확인 불가는 통과가 아니다.

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

### 저평가 근거 — 필수

후보마다 `회사명 밸류에이션 동종업계 PER PBR` / `티커 valuation peers historical multiple`로 추가 검색한다. 최근 30일 기사에서 아래 조건을 모두 확인해야 한다.

1. **정량 비교**: 해당 기업의 평가배수와 유사 기업·업종 또는 자사 과거 정상 범위의 비교 수치가 모두 있다. 기업 값과 비교 값은 모두 유한한 양수여야 한다. PER·EV/EBITDA·PBR은 비교 기준보다 낮아야 하며, FCF 수익률은 비교 기준보다 높아야 한다. 단순한 `저평가` 표현·목표가 괴리·고점 대비 하락률은 인정하지 않는다.
2. **비교 가능성**: 지표 정의, 실제/예상 구분, 대상 결산기 또는 예상 기간, GAAP/조정 기준, 평가 기준일을 확인한다. 비교 대상의 사업·성장률·수익성·부채 차이가 할인 이유가 될 수 있는지 기사 설명으로 점검한다. 시장 전체 평균, 다른 업종, 서로 다른 회계 기준을 임의로 비교하지 않는다. 과거 범위는 산정 기간과 대표값을 확인하며 고평가 시기 최고치만 골라 쓰지 않는다.
3. **현재 종가와의 정합성**: 배수가 직전 거래일 종가 기준인지 확인한다. 과거 기준 PER·PBR은 기사에 기준주가와 EPS·BPS 또는 해당 배수가 있고, 이후 실적·추정치·주식 수 변경이 없음을 확인한 경우에만 `기사 배수 × 직전 종가 ÷ 기사 기준주가`로 보정한다. 원래 값·보정값·전제를 모두 기록한다. EV/EBITDA·FCF 수익률은 주가에 단순 비례시켜 보정하지 않는다. 기준일 정합성을 확인하지 못하면 보류한다.
4. **할인 이유 점검**: 일회성 이익, 경기 고점의 이익, 자산 부실, 낮은 ROE, 과도한 부채나 사업 악화로 낮아 보이는 경우를 검색한다. 그 영향을 반영한 정상 이익·자산 기준의 비교를 확인하지 못하면 보류한다. PBR은 ROE와 자산 건전성 근거를 함께 확인한다. 적자 기업의 음수 PER·EV/EBITDA는 저평가 근거로 사용하지 않는다.

비교·보정·할인율은 Python `Decimal`로 계산하고 반올림 전 값으로 판정한다. 낮을수록 저렴한 배수의 상대 할인율은 `1 − 기업 배수 ÷ 비교 배수`다. 서로 다른 업종·지표의 할인율 크기로 순위를 매기지 않는다. 모든 기업에 `PER 10배 이하`·`PBR 1배 이하` 같은 공통 절대 기준을 적용하지 않는다.

- **저평가 근거 등급 2**: 유사 기업 비교와 자사 과거 정상 범위 비교를 각각 확인하고 위 조건을 모두 통과했다.
- **저평가 근거 등급 1**: 두 비교 중 하나를 확인하고 위 조건을 모두 통과했다.
- **보류**: 둘 다 미확인이거나 비교 가능성·종가 정합성·할인 이유 점검이 미완료다. 검색 요약에 없던 수치를 만들어 채우지 않는다.

### 가치기준 진입가 — 필수

저평가 근거를 통과한 비교 중 **PER 또는 PBR**로 주당 기준가치를 계산할 수 있어야 한다. 위 조건을 통과한 모든 적용 가능한 비교를 수집하고, 가장 높은 배수나 가격만 골라 쓰지 않는다.

| 방식 | 필요한 기사 수치 | 주당 기준가치 |
|---|---|---|
| PER | 해당 기업의 정상화된 또는 비교 가능한 EPS, 유사 기업/자사 과거 정상 PER | `EPS × 비교 PER` |
| PBR | 해당 기업의 BPS, 유사 기업/자사 과거 정상 PBR, ROE·자산 건전성 근거 | `BPS × 비교 PBR` |

- EPS·BPS는 기사에 명시된 값만 쓰며 주가·배수·목표가에서 역산하지 않는다. 예상/실제·결산기·GAAP/조정·통화·보통주/ADR 단위·분할 반영 여부가 비교 근거와 맞아야 한다. 일회성 이익이나 경기 고점 이익을 정상 이익으로 가정하지 않는다.
- 비교배수가 범위로 제시되면 하단을 사용한다. 적용 가능한 여러 기준가치를 모두 계산하고 최솟값을 채택한다. 최근 리포트의 목표가도 확인했다면 그 값과 비교해 더 낮은 값을 **보수 기준가치**로 쓴다. 목표가만으로 기준가치를 만들거나 여러 목표가의 최댓값을 고르지 않는다.
- **가치기준 매수상한 = 보수 기준가치 × 0.80**이다. 20%는 이 루틴의 고정 안전마진으로 수익성이 검증된 최적값이나 손실 방지 장치가 아니다. 근거의 질이 부족한 기업을 마진만 늘려 통과시키지 않는다.
- **최종 진입가 = min(가치기준 매수상한, 직전 거래일 종가)**다. 매수상한과 최종 진입가는 한국 호가 단위·미국 1센트 단위로 각각 내림한다. 가치상한이 높아도 직전 종가보다 높은 가격을 진입가로 제시하지 않는다.
- 종가가 매수상한 이하이면 `가격조건 충족`, 초과하면 `진입대기`로 표시한다. 이는 브리핑 기준 상태이며 실시간 매수 신호가 아니다. 대기 기업도 다른 필수 조건을 모두 통과해야 하며, 진입가 도달 전후 실적·주식 수·추정치·위험·추세 근거가 바뀌면 기존 가격은 폐기한다.
- EV/EBITDA·FCF 수익률만 확인했거나 EPS·BPS/비교배수·통화·기간 등의 근거가 부족하면 **진입가 산출 불가로 보류**한다. 목표가에 일정 할인율을 적용하거나 종가·차트 값으로 대신하지 않는다.
- 기사 발행일·URL·증권사/분석 주체·실적 대상 기간·EPS/BPS·비교배수·단위·각 기준가치·채택 근거·안전마진·매수상한·최종 진입가·상태를 저장한다. 아래 `levels()`에 확인한 근거를 전달해 계산한다.

### 실적 지속성 — 필수

최근 30일 기사에 보도된 최신 발표 결산기의 **실제 이익과 현금 창출 수치**를 확인한다. 아직 실현되지 않은 계약 총액·매출 전망·목표가 상향·주주환원만으로 통과시키지 않는다.

- 일반 기업은 같은 대상 기간의 **영업이익 양수와 영업현금흐름 양수**를 모두 확인한다. 영업현금흐름 대신 정의가 명시된 FCF 양수를 사용할 수 있다. 분기·누적·연간 수치를 섞지 않으며, 최신 분기 수치가 있는데 오래된 연간 흑자로 대체하지 않는다. 일회성 처분이익·평가이익이나 운전자본의 일시적 회수에 의존하면 반복 가능한 수익·현금 창출 근거를 추가 확인한다.
- 금융·보험·리츠처럼 위 지표를 그대로 비교하기 어려운 업종은 해당 업종의 이익과 건전성을 각각 확인한다. 금융·보험은 최신 결산기의 실제 순이익 양수와 적용 규제자본비율의 최소 기준 충족, 리츠는 정의가 명시된 FFO/AFFO 양수와 같은 기간·총액/주당 기준의 `FFO/AFFO ≥ 배당 지급액`을 확인한다. 필요한 수치·비교 기준을 확인하지 못하면 보류하며 일반 기업 기준을 억지로 적용하지 않는다.
- 이익·현금흐름 또는 해당 업종의 대체 지표가 악화된 경우 원인과 일회성 여부를 확인한다. 반복 적자·지속적인 현금 유출이나 구조적인 수익성 악화를 해소할 근거가 없으면 제외한다. 원인·지속성 미확인은 보류한다.
- 신규 데이터센터·공장 등 대규모 투자가 가치평가의 핵심이면 투자금, 확정 자금과 미확정 자금의 구분, 차입·임대료 차감·증자 등 조달 방식 및 비용을 확인한다. 핵심 자금조달이 미확정이거나 그 비용을 반영한 가치평가 근거가 없으면 보류한다. `증자 계획 없음`을 자금 확보로 해석하지 않는다.
- **실적 지속성 등급 2**: 위 조건을 통과하고 같은 기간·기준의 전년 동기 대비 이익과 현금 창출 지표가 모두 유지·개선됐음을 수치로 확인했다. 업종 대체 지표는 이익·건전성이 모두 유지·개선돼야 한다.
- **실적 지속성 등급 1**: 위 조건은 통과했지만 전년 동기 비교 일부가 없거나, 악화 원인과 반복 가능성 검토를 마친 경우다. 설명되지 않은 악화를 등급 1로 통과시키지 않는다.

기사 발행일·URL·실적 대상 기간·실제 수치·일회성 여부·판정 근거를 저장한다. 기사만으로 확인할 수 없으면 기업 수를 줄인다.

### 하락 추세 완화 — 필수

아래 시세 스크립트의 `trend.eligible`이 `true`인 기업만 신규 선정한다. 조건은 다음 두 가지를 모두 충족하는 것이다.

- 직전 거래일 **수정종가 ≥ 최근 60거래일 수정종가 이동평균**.
- 최근 **60거래일 이동평균 ≥ 20거래일 전의 60거래일 이동평균**.

분할·배당을 반영한 동일한 수정주가 계열의 연속된 거래소 세션 80개를 사용한다. 종가 한두 번의 반등·검색 결과의 `바닥`·`반등 기대` 같은 표현으로 대체하지 않는다. `false`는 추세 조건 미충족으로 제외하며, `error`·누락은 확인 미완료로 보류한다. 기사 종가로 가격 블록을 복원해도 추세 조건을 면제하지 않는다.

60일·20일은 이 루틴의 운영 기준이며 수익성이 검증된 최적값이 아니다. 반등 초기를 놓칠 수 있고 통과 후에도 하락할 수 있다. 추세 판정은 기업가치의 증거가 아니다. 가격 블록의 +16%·−8% 비율은 유지하되 기준 가격은 아래에서 계산한 최종 진입가다.

### 제외 조건과 위험 점검

- 주가 상승만으로 긍정 신호를 인정하지 않는다.
- 시세 조회의 `surge_excluded`가 `true`이거나, 72시간 기사에서 한국 하루 15% 이상·미국 20% 이상 상승 또는 상한가가 확인되면 제외한다. `surge_excluded`는 반올림 전 가격으로 계산하며, 표시용 `daily_change_pct`로 판정하지 않는다.
- 아래 **위험 검색을 마친 후보만** 확정한다. 대체 후보도 동일하다.

후보마다 회사명(가능하면 종목코드·티커 포함)과 30일 기간을 지정해 네 묶음을 각각 WebSearch한다. 한국은 한국어, 미국은 영어로 검색한다.

| 묶음 | 한국어 | 영어 |
|---|---|---|
| 실적·전망 | `실적 하회 OR 컨센서스 하회 OR 가이던스 하향` | `earnings miss OR revenue miss OR guidance cut OR lowered outlook` |
| 희석·규제·소송 | `유상증자 OR 전환사채 OR 규제 OR 소송` | `share offering OR convertible notes OR regulatory action OR lawsuit` |
| 이익 추정치 | `이익 추정치 하향 OR EPS 하향 OR 실적 전망 하향` | `earnings estimate cut OR EPS revision down OR earnings forecast lowered` |
| 현금·자금조달 | `영업현금흐름 악화 OR 자금조달 OR 차입 부담` | `negative operating cash flow OR financing risk OR funding gap` |

- **제외**: 30일 기사에서 해당 기업의 실적 하회·가이던스 하향·희석 공시·규제 조치·소송을 확인한 경우. 업종 제도 설명이나 일반적인 가능성만으로 제외하지 않는다.
- **추정치 하향 확인 시 보류**: 증권사·발표일·동일 지표와 기간의 기존 값·새 값을 확인한다. 하향 이후 같은 기간·기준의 추정치 유지·상향과 하향된 이익을 반영한 새 저평가 근거를 모두 확인한 경우에만 재검토한다. 주가가 더 많이 하락했다는 이유로 통과시키지 않는다. 회사 가이던스 하향은 위 제외 규칙을 따른다.
- **현금·자금조달 위험**: 일반적인 가능성만으로 제외하지 않는다. 실제 지속적인 현금 유출·구조적 악화는 실적 지속성 규칙으로 제외하고, 핵심 자금조달 미확정·비용 미반영은 보류한다. 키워드만으로 판정하거나 계약 총액을 순이익·기업가치로 대체하지 않는다.
- **위험 점검 미완료**: 관련 악재의 발행일·대상·사건을 추가 검색해도 확인하지 못하거나 필수 검색이 실패한 경우. 이 후보는 확정하지 않고 다른 후보를 계속 점검한다.
- **해당 검색에서 제외·보류 사유 미발견**: 네 검색을 정상 완료했으나 해당 근거를 찾지 못한 경우. 이를 `위험 없음`·`안전`으로 표현하지 않는다. 검색에서 악재를 찾지 못한 것과 실적 지속성의 긍정적인 수치 확인은 별개다.

검색어 네 묶음·기간·판정·근거는 최종 실행 기록에 남긴다. 발견한 기사에는 URL·발행일·사건 요약을 기록한다.

### 상승여력과 선정 순서

상승여력은 목표주가 ÷ 직전 거래일 종가 − 1인 보조 지표이며 저평가 판정이 아니다. **가격 계산**의 `levels()`는 `Decimal`로 계산하며, 15% 충족 여부는 `upside_eligible`로 판단한다. 표시용 `upside_pct`로 판정하지 않는다.

- 목표가를 확인한 경우에는 기존 기준대로 **상승여력 15% 미만이면 제외**한다. 15% 이상이어도 저평가 근거를 대신하지 못한다.
- 목표가가 미확인이어도 필수 여섯 조건을 모두 통과하면 같은 후보군에서 비교한다. 종가가 없으면 추세·가치평가의 정합성을 통과시킬 수 없으므로 신규 선정하지 않는다.
- 상승여력 가점과 `총점`은 사용하지 않는다. 후보마다 **저평가 근거 등급·실적 지속성 등급·신호 점수**를 별도로 기록한다.
- 시장별로 모든 필수 조건을 통과한 후보 안에서 **저평가 근거 등급 → 실적 지속성 등급 → 가격조건 충족 우선 → 신호 점수 → 이미 고른 기업과 다른 업종 → 주요 매체 수 → 상승여력 원값** 순으로 비교한다. 첫 기업은 업종 비교를 생략한다. 마지막 동률 비교에서만 목표가 확인 후보를 우선하고 `Decimal(upside_raw)`를 비교하며, 그래도 같으면 종목코드/티커 오름차순으로 정한다. 대기 기업은 부모·답글에 `진입대기`를 명시한다.
- 필수 근거가 미확인·보류인 후보는 순위에 넣거나 빈자리 충원에 사용하지 않는다. 적격 기업이 없으면 빈 섹션 형식으로 게시한다.
- 시가총액은 규모 참고용이다. 점수·동점 처리·제외 조건이나 저평가·안정성 판단에 쓰지 않는다.

## 중복 방지와 재실행

채널의 `📈 Daily Stock Brief` 제목과 날짜를 기준으로 부모 메시지를 식별한다. 도구가 돌려주는 굵게 표기 유무는 무시한다.

- 최근 7일 부모 메시지 전부에서 기업을 확인해 제외한다. 이때 스레드와 사람이 쓴 다른 메시지는 읽지 않는다.
- 오늘 안에서도 같은 기업을 두 번 쓰지 않는다.
- **오늘 부모 메시지가 있으면 새로 만들지 않고 해당 스레드만 읽는다.** 한국·미국 섹션 답글이 모두 있으면 아무것도 게시하지 않고 종료한다. 분할 게시였다면 이어지는 답글까지 완성됐는지 확인한다.
- 빠진 섹션은 부모에 적힌 기업·순서로만 작성하고 네 묶음의 위험 점검을 다시 한다. 저평가·실적 지속성·추세의 원래 근거와 판정을 스냅샷에서 복원하고 현재 지침의 필수 조건도 충족하는지 확인한다. 분할 답글 일부가 이미 있으면 게시된 항목은 건너뛰고 누락 항목만 이어 쓴다. 제외·미완료 기업은 같은 위치에 이름과 보류 사유만 쓰며 대체 기업·가격 블록은 넣지 않는다. 이 보류 표시는 일반 기업 항목·빈 섹션 형식의 예외다.
- 이전 형식 스냅샷에 필수 근거가 없으면 원래 세션 기록에서 브리핑 기준 시각 이전 자료와 판정을 모두 확인한 경우에만 복원한다. 복원할 수 없으면 `회사명 — 보류: 부모 작성 당시 저평가·실적 지속성·추세 근거 복원 불가`로 쓴다. 새 가격·추세·가치평가로 과거 부모의 적격성을 대체하지 않는다. 이미 완성된 오늘 게시물은 수정·재게시하지 않는다.
- 가격은 아래 **게시 전 스냅샷**에서 복원한다. 스냅샷이 없으면 원래 세션 기록이나 이미 게시된 해당 기업 답글에서 종가·목표가·출처·기준일과 가치 계산의 모든 입력·가격 규칙 버전을 확인한 경우에만 복원한다. 목표가의 원래 미확인 상태도 보존한다.
- 원래 가격의 값·미확인 상태를 복원하지 못하면 해당 위치에 `회사명 — 보류: 부모 작성 당시 가격 근거 복원 불가`만 쓴다. 새 시세·목표가·EPS/BPS·비교배수로 대체하거나 부모의 반올림된 상승여력에서 가격을 역산하지 않는다. `value-entry-v1`의 원래 입력을 모두 복원했으면 아래 `levels()`로 다시 계산해 저장된 기준가치·매수상한·진입가·상태·손절·수익실현·부모 상승여력과 대조한다. 불일치하거나 적격 조건을 통과하지 못해도 보류한다. 기간은 부모 날짜 + 28일이다.
- 이전 종가 기준 방식의 스냅샷은 새 방식으로 재계산하지 않는다. 미게시 기업 항목은 `회사명 — 보류: 부모 작성 당시 가치기준 진입가 근거 없음`으로 쓴다. 새 규칙에 필요한 입력 또는 가격 규칙 버전을 복원하지 못한 경우도 같다. 이미 완성된 게시물은 그대로 둔다.

### 게시 전 스냅샷

부모를 보내기 전에 검증을 마친 내용을 작업 폴더의 `.brief-state/YYYY-MM-DD.json`에 UTF-8 JSON으로 저장한다. 가격·비율은 소수 문자열, 날짜는 `YYYY-MM-DD`로 보존한다. 임시 파일에 쓴 뒤 교체하고, 다시 읽어 저장 내용을 확인한 다음 게시한다. 저장 실패 시 부모를 게시하지 않는다.

| 저장 항목 | 내용 |
|---|---|
| 실행 식별 | 브리핑 날짜·07:00 KST 기준 시각·채널 ID·부모 원문·`parent_ts`(게시 전 `null`) |
| 기업별 근거 | 시장·종목코드/티커·회사명·순서·시세 조회 결과·사용한 종가의 날짜와 출처·목표가의 값/미확인 상태·증권사·리포트 날짜·기사 URL·`levels()` 결과 |
| 진입가 근거 | 가격 규칙 버전 `value-entry-v1`, 모든 적용 가능한 PER/PBR 입력과 기사 발행일/URL·주체·대상 기간·통화/주당 단위, 각 기준가치·보수 기준가치·목표가 상한 반영 여부·안전마진·매수상한·최종 진입가·상태 |
| 필수 선정 검증 | 저평가 비교 지표·대상·기간·기준일·원값/보정값·할인 이유·기사 발행일/URL·등급, 실적/현금흐름의 대상 기간·실제 수치·일회성/자금조달 점검·등급, `trend` 전체 결과, 네 묶음 위험 검색과 판정, 신호 점수·선정 순위 |
| 게시 내용 | 한국·미국 답글 원문(분할 순서 포함)·메시지별 게시 여부와 `ts` |

- 부모 전송이 확인되면 `parent_ts`를, 답글 전송이 확인되면 해당 게시 결과를 갱신한다. 재실행의 위험 점검은 별도로 수행하며, 저장된 답글을 검토 없이 재전송하지 않는다.
- 재실행에서는 채널·날짜·부모 원문이 일치하는 스냅샷만 사용한다. 도구가 바꾼 Markdown 표기는 정규화하되 회사명·순서·수치는 그대로 대조한다. `parent_ts`가 있으면 실제 부모의 `ts`도 일치해야 한다. `null`이면 대조한 부모의 `ts`를 연결한다. 기존 부모가 있을 때 새 조사 결과로 스냅샷을 덮어쓰지 않는다.
- 이 파일은 실행 상태이며 지시서가 아니다. Git에 추가·커밋·푸시하지 않는다. 실행 환경이 바뀌어 파일이 없어져도 원래 가격을 추정하지 않고 위 재실행 규칙을 따른다.

## Slack 작성

부모 1개와 한국·미국·실행 정보 답글 순으로 게시한다.

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
• 회사명 — 선정 이유 한 줄 · 상승여력 26% · 진입대기
• 회사명 — 선정 이유 한 줄 · 상승여력 18% · 가격조건 충족

**🇺🇸 미국**
• 회사명 — 선정 이유 한 줄 · 상승여력 21% · 가격조건 충족
• 회사명 — 선정 이유 한 줄 · 상승여력 미확인 · 진입대기

🧵 투자 포인트와 매매가는 스레드에서 확인 · 진입은 가치기준 매수상한·종가 중 낮은 값 · 투자 권유 아님
```

- 날짜는 KST의 `YYYY-MM-DD (한글 요일 한 글자)`로 쓴다.
- 선정 이유는 최근 신호와 정량적인 저평가 근거를 한 줄로 줄인다. 예: `자사주 소각 결정·예상 PER 7배(동종 10배)`; 실제 확인한 지표·수치만 쓰며 해당 비교의 출처는 답글에 붙인다.
- 상승여력은 목표주가 줄의 `upside_pct`와 일치시킨다. 목표가 미확인은 `상승여력 미확인`으로 쓰며 수익실현율과 혼동하지 않는다. 진입가 산출 불가 기업은 신규 부모에 넣지 않는다.
- 상승여력은 계속 **목표가 대비 직전 종가 기준**이며 진입가 기준으로 바꾸지 않는다. 각 기업 줄 끝에 `가격조건 충족` 또는 `진입대기`를 붙여 구분한다.

### 기업 답글

번호는 시장마다 1부터 시작한다. 다음은 형식 예시다.

```
🇰🇷 **한국 기업**
**1. 회사명 (005930)**
_회사 설명 1줄 · 시가총액 12.3조 원 (네이버, 11/2 07:02 KST 조회)_
• 최근 무슨 일이 있었나
• 저평가 비교 수치와 실적·현금 창출 근거
• 리스크 또는 다음에 확인할 것
📊 추세: 수정종가 ≥ 60일 평균 · 60일 평균 ≥ 20거래일 전 평균 (Yahoo Finance, 10/30 기준)
💵 가격 (10/30 종가 71,200원, 네이버 · 기간 11/30까지)
• 목표주가 90,000원 (A증권, 10/30 상향 · 상승여력 26%)
• 가치기준 매수상한 64,000원 (2027년 예상 EPS 8,000원 × 비교 PER 10배 = 80,000원 · 안전마진 20% · A증권 10/30)
• 수익실현 74,200원 (+16%)
• 진입 64,000원 이하 (진입대기)
• 손절 58,900원 (−8%)
🔗 [매체명](https://원문링크)

**2. ...**
```

- **제목**: 한국은 6자리 종목코드, 미국은 티커를 검색 결과로 확인해 괄호에 넣는다. 확인하지 못하면 괄호를 생략하고 시가총액은 미확인으로 쓴다.
- **설명 1줄**: 무엇을 만들거나 파는 회사인지 한 문장 뒤에 `market_cap.display`를 붙인다. 시가총액은 한국 원화·미국 달러이며 조·억·만 단위로 소수 첫째 자리까지 반올림하고 `.0`을 생략한다. 출처·KST 조회 시각은 종가 기준일이나 데이터 갱신 시각으로 해석하지 않는다. 실패 시 기사·주식 수 계산으로 보충하지 않는다.
- **투자 포인트는 정확히 3줄**, 각 한 문장으로 쓴다.
  1. **사건**: 인정된 신호의 사건일·수치. 실적은 지표·기간·실제치·컨센서스, 가이던스는 기존·새 값, 자사주는 결정·이행 구분을 밝힌다.
  2. **저평가·실적 지속성**: 비교 가능한 평가배수와 기준 수치, 실적·현금 창출의 실제 근거를 한 문장으로 쓴다. 지표·대상 기간을 밝히고 과거 배수 보정 시 보정값임을 표시한다. 이 근거의 기사 발행일·링크를 붙이며 목표가 괴리만으로 저평가라고 쓰지 않는다.
  3. **리스크·다음 확인**: 기사에 나온 약점·반론·일회성 요인을 우선한다. 없으면 무엇이 확인될 때 판단이 달라지는지 조건으로 쓴다. 확인 일정은 기사에 있는 것만 쓴다. 제외 사유가 있는 기업을 이 줄로 설명해 통과시키지 않는다.
- **추세 확인 줄**: 포인트 3줄 아래에 위 예시처럼 `trend`의 두 조건·출처·기준일을 표시한다. 수정주가 계열의 통과 사실이며 향후 상승을 예측하지 않는다. 분할 시 이 줄과 가격 블록을 해당 기업 항목에 함께 둔다.

### 가격 블록

분석 기업마다 설명·포인트 아래에 작성한다. 차트·지지선·저항선으로 공식을 바꾸지 않는다.

제목은 `💵 가격 (기준일 종가 금액, 출처 · 기간 M/D까지)`다.
기준일·출처·종가 금액은 시세의 `date`·`source`·`close`다. 종가와 진입가를 구분한다. 가치기준 매수상한 줄에는 채택한 EPS/BPS·비교배수·대상 기간·주체·리포트 날짜·기사 링크와 안전마진을 붙인다. 여러 방식의 최솟값이면 그 사실을 적고 전체 산식은 실행 기록에 남긴다. 기간은 브리핑 날짜 + 28일이며, 그 이후에는 이 참고값을 사용하지 않는다는 기준이다.

가격 줄은 다음 순서를 유지한다. 한국은 원, 미국은 달러(`$182.40`)로 쓴다.

| 줄 | 값·표기 |
|---|---|
| 목표주가 | 최근 증권사·애널리스트 리포트를 보도한 기사 하나의 목표가, 증권사·리포트 날짜·상향/하향 여부·상승여력 `upside_pct` |
| 가치기준 매수상한 | `buy_ceiling`과 보수 기준가치·산식·안전마진 20%·근거 |
| 수익실현 | `min(보수 기준가치, 진입가 × 1.16)`. 보수 기준가치에는 확인한 목표가 상한도 반영돼 있다. 괄호는 `(+N%)`로 `take_profit_pct`만 표시 |
| 진입 | `entry` 뒤에 `이하`와 `entry_status`를 붙임. 종가를 그대로 복사하지 않음 |
| 손절 | 진입가 × 0.92, 괄호는 `(−8%)` |

- 목표가는 보통 6~12개월 기준이므로 4주 수익실현가로 그대로 쓰지 않는다. 선정 기사에 없으면 `회사명 목표주가` / `티커 price target`으로 추가 검색한다. 30일 안의 기사로 확인하지 못하면 `• 목표주가 미확인`으로 쓰고, 컨센서스 평균·오래된 값으로 채우지 않는다. +16%·−8%는 공통 계산 규칙이며 적정가치·기대수익률이 아니다. 28일은 참고가격의 유효기간이며 저평가 해소 예상 기간이 아니다.
- 한국은 KRX 호가 단위, 미국은 1센트 단위로 매수상한·진입·수익실현은 내림하고 손절은 올림한다. 손절·수익실현 비율의 분모는 최종 진입가다. 실제 체결가가 다르면 이 비율도 달라지므로 체결가 기준으로 재산정해야 하며, 루틴은 주문하거나 체결을 가정해 기록하지 않는다. **반드시 아래 코드로 계산**한다.

#### 종가 조회 실패 시

신규 선정에서는 기사 종가로 복원하더라도 별도의 `trend.eligible == true`와 저평가 기준일 정합성·가치기준 진입가 입력을 반드시 확인한다. 직전 종가나 진입가 근거를 끝내 확인하지 못하면 신규 선정하지 않는다. 재실행에서도 가격 근거가 없으면 가격 블록 대신 보류 문구만 쓴다.

최상위 `error`가 있으면 기사 가격을 확인한다. 선정 기사에 없으면 `회사명 주가` / `티커 stock price`로 한 번 더 검색해 72시간 안의 기사에서 찾는다.

스크립트의 `expected_date`가 있으면 기사 종가의 거래일도 그 날짜와 같아야 한다. 거래소 달력 조회가 실패해 `expected_date`가 없으면 시장별 휴장·거래일을 WebSearch로 확인하고 근거를 기록한다. 직전 거래일 자체를 확인하지 못하면 매매가를 산출하지 않는다. 후보 간 날짜 비교나 주말만 제외하는 계산으로 대체하지 않는다.

- **직전 거래일 종가 확인**: 그 값으로 계산하고 가격 제목의 출처를 매체명으로 쓴다.
- **확인 불가**: 신규 선정하지 않고 실행 기록에 종가/진입가 산출 불가 사유를 남긴다. 기존 부모의 미게시 항목은 해당 위치에 회사명과 보류 사유만 쓴다.

장중·오래된 종가·추정값으로 매매가를 계산하지 않으며, 조회 오류는 최종 기록에 남긴다.

### 가격 계산

```python
from datetime import date, timedelta
from decimal import Decimal, ROUND_CEILING, ROUND_FLOOR, ROUND_HALF_UP

D = Decimal
ENTRY_MARGIN = D("0.20")
PRICE_RULE_VERSION = "value-entry-v1"

def price_value(value):
    p = D(str(value))
    if not p.is_finite() or p <= 0:
        raise ValueError("가격은 유한한 양수여야 함")
    return p

def percent(value):
    return int(value.quantize(D("1"), rounding=ROUND_HALF_UP))

def krx_tick(p):  # KRX 호가 단위 (유가증권·코스닥 공통)
    for limit, t in [(2000, 1), (5000, 5), (20000, 10), (50000, 50), (200000, 100), (500000, 500)]:
        if p < limit:
            return D(t)
    return D(1000)

def reference_values(valuation_bases):
    if not isinstance(valuation_bases, list) or not valuation_bases:
        raise ValueError("PER/PBR 기반 진입가 근거 필요")
    results = []
    for basis in valuation_bases:
        if not isinstance(basis, dict) or basis.get("method") not in {"PER", "PBR"}:
            raise ValueError("주당 기준가치 계산은 PER 또는 PBR만 허용")
        if not {"per_share", "benchmark"} <= basis.keys():
            raise ValueError("기사의 EPS/BPS와 비교배수 입력 필요")
        per_share = price_value(basis["per_share"])
        benchmark = price_value(basis["benchmark"])
        value = price_value(per_share * benchmark)
        results.append({**basis, "per_share": str(per_share), "benchmark": str(benchmark),
                        "reference_value": str(value)})
    return results

def round_price(value, market, rounding):
    tick = krx_tick(value) if market == "KR" else D("0.01")
    result = (value / tick).to_integral_value(rounding=rounding) * tick
    return price_value(result)

# valuation_bases: validated article inputs, each with method, per_share and benchmark.
# per_share is EPS for PER, BPS for PBR; preserve source/period/currency metadata as well.
def levels(price, target, market, brief_date, valuation_bases):
    if market not in {"KR", "US"}:
        raise ValueError("시장은 KR 또는 US여야 함")
    price = price_value(price)
    target = price_value(target) if target is not None else None
    values = reference_values(valuation_bases)
    valuation_min = min(D(v["reference_value"]) for v in values)
    conservative_value = min(valuation_min, target) if target is not None else valuation_min
    ceiling_raw = conservative_value * (1 - ENTRY_MARGIN)
    buy_ceiling = round_price(ceiling_raw, market, ROUND_FLOOR)
    entry = round_price(min(price, buy_ceiling), market, ROUND_FLOOR)
    stop = round_price(entry * D("0.92"), market, ROUND_CEILING)
    tp = round_price(min(conservative_value, entry * D("1.16")), market, ROUND_FLOOR)
    if not stop < entry < tp:
        raise ValueError("호가 반올림 후 손절 < 진입 < 수익실현 조건 미충족")
    upside = target / price - 1 if target is not None else None
    return {"price_rule_version": PRICE_RULE_VERSION, "valuation_values": values,
            "valuation_min": str(valuation_min), "conservative_value": str(conservative_value),
            "target_capped": target is not None and target < valuation_min,
            "entry_margin": str(ENTRY_MARGIN), "buy_ceiling_raw": str(ceiling_raw),
            "buy_ceiling": str(buy_ceiling), "entry": str(entry),
            "entry_status": "가격조건 충족" if price <= buy_ceiling else "진입대기",
            "stop": str(stop), "take_profit": str(tp),
            "take_profit_pct": percent((tp / entry - 1) * 100),
            "upside_raw": str(upside) if upside is not None else None,
            "upside_eligible": target >= price * D("1.15") if target is not None else None,
            "upside_pct": percent(upside * 100) if upside is not None else None,
            "until": (brief_date + timedelta(days=28)).strftime("%-m/%-d")}

# Illustrative inputs only; actual runs require verified article evidence.
print(levels(71200, 90000, "KR", date(2026, 11, 2),
             [{"method": "PER", "per_share": "8000", "benchmark": "10"}]))
```

## 시세 조회

후보를 확정하기 전에 아래 명령에 `KR:종목코드` / `US:티커`를 한 번에 넘긴다. 종목마다 JSON 한 줄을 출력한다.

- 종가·등락률은 한국 네이버 우선, 실패 시 Yahoo Finance(`.KS` → `.KQ`), 미국은 Yahoo Finance다. 각 출처의 마지막 거래일이 거래소 달력의 `expected_date`와 일치해야 성공이다. 오래된 응답은 오류로 기록하고 다음 출처를 조회한다.
- 시가총액은 한국 네이버 시세 API, 미국 Yahoo Finance의 `marketCap` 제공값이다. 종가 조회와 독립 처리하므로 종가·달력 조회가 실패해도 시가총액을 조회한다.
- `expected_date`는 브리핑 날짜 전날까지의 최근 거래일이다. `exchange_calendars`의 한국 `XKRX`·미국 `XNYS` 달력을 사용한다. 미국 NYSE·Nasdaq에는 같은 거래일 달력을 적용한다. 달력 조회 실패 시 종가를 성공으로 처리하지 않고 위 **종가 조회 실패 시**를 따른다.
- 내부 날짜는 `YYYY-MM-DD`이며, 메시지에서는 `M/D`로 표시한다. `daily_change_pct`는 브리핑 날짜 − 3일 이후의 일간 등락률을 소수 둘째 자리로 반올림한 표시값이다. `surge_excluded`는 원래 가격을 `Decimal`로 비교한 결과다.
- 대체 출처로 성공해도 `warnings`에 남은 이전 출처 오류를 최종 기록에 포함한다.
- `market_cap`은 원/달러 값(`value`), 통화(`currency`), 출처(`source`), KST 조회 시각(`retrieved_at`), 표시 문구(`display`)다. `market_cap.error`만 있으면 종가 조회 실패로 취급하지 않고 `시가총액 미확인`을 표시한다.
- 추세는 종가 조회와 독립적으로 Yahoo Finance의 `Adj Close`에서 계산한다. 한국은 `.KS` → `.KQ`, 미국은 티커를 사용한다. 조회 범위는 과거 400일이며 거래소의 마지막 80세션이 모두 있어야 한다. 서로 다른 출처·수정/비수정 계열을 이어 붙이지 않는다. 네이버 종가 성공도 추세 조회를 생략할 이유가 되지 않는다.
- `trend`는 출처·심볼·기준일·표본 시작일·20거래일 전 비교일·수정종가·두 이동평균·조건별 판정·`eligible`·사용한 80개 수정종가를 담는다. 판정은 반올림 전 `Decimal` 합계로 하며 이동평균의 표시 반올림값을 쓰지 않는다. 추세 오류는 `trend.error`, 이전 대체 출처 오류는 `trend.warnings`에 남긴다. 종가·시가총액 성공과 별개로 처리한다.

아래 명령은 Python 3.10 이상에서 설치·실행에 같은 인터프리터를 쓴다. 설치 실패는 기록하고 조회 스크립트는 계속 실행해 이용 가능한 결과를 남긴다.

```bash
python3 -m pip install -q yfinance 'exchange_calendars>=4.11.3'
# 첫 인수는 실행 시작 시 고정한 브리핑 날짜로, 나머지는 실제 후보로 바꾼다.
python3 - 2026-11-02 KR:005930 US:NVDA <<'EOF'
import ast, json, sys, urllib.request
from datetime import date, datetime, timedelta, timezone
from decimal import Decimal, ROUND_HALF_UP
from functools import lru_cache

KST = timezone(timedelta(hours=9))
today = date.fromisoformat(sys.argv[1])  # 실행 시작 시 고정한 브리핑 날짜
window_start = today - timedelta(days=3)  # 수집 기간이 걸친 첫 날짜

@lru_cache(maxsize=2)
def trading_calendar(market):
    import exchange_calendars as xcals
    if tuple(int(p) for p in xcals.__version__.split(".")[:3]) < (4, 11, 3):
        raise ValueError("exchange_calendars 4.11.3 이상 필요")
    name = {"KR": "XKRX", "US": "XNYS"}[market]
    # 일요일·연휴에도 조회 날짜가 달력의 마지막 세션을 넘지 않게 여유를 둔다.
    return xcals.get_calendar(name, start=today - timedelta(days=400), end=today + timedelta(days=14))

def expected_session(market):
    calendar = trading_calendar(market)
    return calendar.date_to_session(today - timedelta(days=1), direction="previous").date()

def price_value(value):
    p = Decimal(str(value))
    if not p.is_finite() or p <= 0:
        raise ValueError("가격은 유한한 양수여야 함")
    return p

def naver(code):
    url = (f"https://api.finance.naver.com/siseJson.naver?symbol={code}&requestType=1&timeframe=day"
           f"&startTime={today - timedelta(days=30):%Y%m%d}&endTime={today:%Y%m%d}")
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
    text = urllib.request.urlopen(req, timeout=10).read().decode("utf-8", "replace")
    return [(datetime.strptime(r[0], "%Y%m%d").date(), r[4]) for r in ast.literal_eval(text.strip())[1:] if r[4]]

def yahoo(symbol):
    import yfinance as yf
    h = yf.Ticker(symbol).history(start=(today - timedelta(days=30)).isoformat(),
                                  end=today.isoformat(), auto_adjust=False, timeout=10)
    return [(ts.date(), Decimal(str(c)).quantize(Decimal("0.01"), rounding=ROUND_HALF_UP))
            for ts, c in h["Close"].dropna().items()]

def trend_from_rows(rows, session_dates):
    # 두 60세션 구간은 20세션 떨어져 있으므로 연속 80세션이 필요하다.
    dates = list(session_dates)[-80:]
    if len(dates) != 80 or dates != sorted(set(dates)):
        raise ValueError("추세 계산에 필요한 연속 거래소 세션 80개 없음")
    rows = list(rows)
    row_dates = [d for d, _ in rows]
    if not rows or len(set(row_dates)) != len(row_dates):
        raise ValueError("추세 시세 없음 또는 거래일 중복")
    if max(row_dates) != dates[-1]:
        raise ValueError(f"추세 기준일 불일치: {max(row_dates)} (기대: {dates[-1]})")
    prices = dict(rows)
    missing = [d.isoformat() for d in dates if d not in prices]
    if missing:
        raise ValueError(f"추세 거래일 데이터 누락: {', '.join(missing[:3])}")
    adjusted = [price_value(prices[d]) for d in dates]
    current_sum = sum(adjusted[-60:], Decimal(0))
    previous_sum = sum(adjusted[:60], Decimal(0))
    close_ok = adjusted[-1] * 60 >= current_sum
    average_ok = current_sum >= previous_sum
    return {"date": dates[-1].isoformat(), "sample_start": dates[0].isoformat(),
            "comparison_date": dates[-21].isoformat(), "data_points": 80,
            "adjustment": "Yahoo Finance Adj Close (분할·배당 반영)",
            "adjusted_close": str(adjusted[-1]),
            "ma60": str(current_sum / 60),
            "ma60_20_sessions_ago": str(previous_sum / 60),
            "close_ge_ma60": close_ok, "ma60_non_decreasing": average_ok,
            "eligible": close_ok and average_ok,
            "adjusted_closes": [{"date": d.isoformat(), "close": str(p)}
                                for d, p in zip(dates, adjusted)]}

def trend(market, code):
    base = {"source": "Yahoo Finance", "expected_date": None}
    errors = []
    try:
        expected = expected_session(market)
        base["expected_date"] = expected.isoformat()
        calendar = trading_calendar(market)
        dates = [s.date() for s in calendar.sessions_in_range(
            today - timedelta(days=400), expected)][-80:]
        symbols = [f"{code}.KS", f"{code}.KQ"] if market == "KR" else [code.replace(".", "-")]
        for symbol in symbols:
            try:
                import yfinance as yf
                h = yf.Ticker(symbol).history(start=(today - timedelta(days=400)).isoformat(),
                                              end=today.isoformat(), auto_adjust=False,
                                              back_adjust=False, timeout=10)
                if "Adj Close" not in h:
                    raise ValueError("수정종가 Adj Close 없음")
                rows = [(ts.date(), value) for ts, value in h["Adj Close"].items() if ts.date() < today]
                result = trend_from_rows(rows, dates)
                # 정상 데이터로 false이면 다른 심볼로 유리한 판정을 찾지 않는다.
                return {**base, "symbol": symbol, **result, "warnings": errors}
            except Exception as e:
                errors.append(f"{symbol}: {type(e).__name__}: {e}"[:200])
    except Exception as e:
        errors.append(f"추세 달력: {type(e).__name__}: {e}"[:200])
    return {**base, "eligible": None, "error": errors}

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
    base = {"market": market, "code": code, "expected_date": None}
    try:
        expected = expected_session(market)
        base["expected_date"] = expected.isoformat()
    except Exception as e:
        return {**base, "error": [f"거래소 달력: {type(e).__name__}: {e}"[:200]]}
    if market == "KR":
        sources = [("네이버", lambda: naver(code)), ("Yahoo Finance", lambda: yahoo(f"{code}.KS")),
                   ("Yahoo Finance", lambda: yahoo(f"{code}.KQ"))]
    else:
        sources = [("Yahoo Finance", lambda: yahoo(code.replace(".", "-")))]
    errors = []
    for name, fetch in sources:
        try:
            rows = sorted((d, price_value(c)) for d, c in fetch() if d < today)
            if len(rows) < 2:
                raise ValueError("종가·등락률 계산에 필요한 데이터 없음")
            if rows[-1][0] != expected:
                raise ValueError(f"종가 기준일 불일치: {rows[-1][0]} (기대: {expected})")
            # 휴장일을 넘긴 정상 등락과 거래일 데이터 누락을 구분한다.
            prices = dict(rows)
            calendar = trading_calendar(market)
            sessions = calendar.sessions_in_range(window_start, expected) if window_start <= expected else []
            changes, surge = {}, False
            threshold = Decimal("1.15") if market == "KR" else Decimal("1.20")
            for session in sessions:
                d = session.date()
                previous = calendar.previous_session(session).date()
                if d not in prices or previous not in prices:
                    raise ValueError(f"일간 등락률 거래일 데이터 누락: {previous} / {d}")
                p, c = prices[previous], prices[d]
                changes[d.isoformat()] = str(((c / p - 1) * 100).quantize(
                    Decimal("0.01"), rounding=ROUND_HALF_UP))
                surge = surge or c >= p * threshold
            return {**base, "source": name, "date": expected.isoformat(),
                    "close": str(rows[-1][1]), "daily_change_pct": changes,
                    "surge_excluded": surge, "warnings": errors}
        except Exception as e:
            errors.append(f"{name}: {type(e).__name__}: {e}"[:200])
    return {**base, "error": errors}

for arg in sys.argv[2:]:
    market, code = arg.split(":", 1)
    result = quote(market, code)
    result["market_cap"] = market_cap(market, code)
    result["trend"] = trend(market, code)
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

1. **기준 설정**: 브리핑 날짜를 고정하고 72시간·30일·최근 7일의 기간을 계산한다. 추세 표본은 별도로 직전 거래일까지의 80거래일이다.
2. **채널 확인**: ID `C0CDFHG8PL7`로 읽고, 실패하면 `slack_search_channels`로 `daily-stock-brief`를 찾는다. 중복 목록을 만들고, 오늘 부모가 있으면 **중복 방지와 재실행** 규칙을 따른다.
3. **후보 조사**: 신규 신호·발행일·사건일·인정 근거를 확인한다. 저평가 비교와 실적 지속성의 실제 수치를 수집하고 후보별 네 묶음 위험 점검을 수행한다.
4. **후보 확정**: 남은 후보의 시세·추세·목표가를 조회한다. 종가와 저평가 비교 기준일을 맞추고, 저평가·실적 지속성·위험·급등·추세·상승여력 조건을 모두 검증한다. 기사로 확인한 모든 적용 가능한 PER/PBR 입력을 `levels()`에 전달해 가치기준 매수상한·최종 진입가·상태와 손절·수익실현을 계산한다. 산출 불가 후보는 보류하고 필수 조건 통과 후보만 정해진 순서로 비교한다. 대체 후보도 같은 절차를 거치며 부족하면 빈자리로 둔다.
5. **작성·검증**: 부모와 기업 답글을 작성하고 아래 게시 전 점검을 통과시킨 뒤 스냅샷을 저장·확인한다.
6. **직접 게시**: `slack_send_message`로 부모를 보내고 응답의 `ts`를 `thread_ts`로 써서 한국·미국 순으로 답글을 보낸다. 각 게시 결과를 스냅샷에 기록한다. 초안 도구는 쓰지 않는다. `reply_broadcast`·`unfurl_app_links`는 설정하지 않는다.
7. **마무리**: 실행 정보를 집계해 마지막 답글로 보내고, 아래 최종 결과를 남긴다.

### 게시 전 점검

게시한 메시지는 수정하지 않는다. 틀린 곳은 전송 전에 고친다.

- **적격성**: 날짜·중복·신호 인정·정량 저평가·실적 지속성·네 묶음 위험 점검·급등·추세·가치기준 진입가 산출·상승여력 조건을 통과했으며 등급·가격상태·신호 점수·순위와 기록이 일치하는가. 미확인 후보로 충원하지 않았는가.
- **내용**: 설명 1줄·포인트 3줄·추세 확인 줄이 정해진 순서이고, 저평가 비교와 실제 이익·현금 창출의 근거 및 발행일·링크가 있는가. 모든 사실·수치·전망 인용에 근거가 있으며 권유·예측 문장이 없는가.
- **가격**: 종가 거래일이 `expected_date` 또는 검색으로 확인한 직전 거래일과 같은가. EPS/BPS·비교배수의 기간·회계·통화·주당 단위·출처가 맞는가. 모든 적용 가능한 기준가치의 최솟값·목표가 상한·20% 안전마진을 적용했는가. `entry ≤ buy_ceiling` 및 `entry ≤ close`, `stop < entry < take_profit`인가. `levels()` 결과·호가 단위·기간이 맞으며 손익 비율은 진입가 기준, 상승여력은 종가 기준인가. 산출 불가 기업을 보류했는가.
- **추세·비교**: `trend.date == expected_date`이고 연속 80세션의 동일 수정주가 계열로 `trend.eligible == true`를 확인했는가. 기사 종가로 추세를 대체하지 않았는가. 가치평가 배수의 실제/예상·기간·회계 기준·종가 기준일 및 허용된 보정이 맞는가.
- **일치**: 부모·답글·스냅샷의 회사명·순서·상승여력·가격조건 충족/진입대기 상태가 같은가. 재실행에서 원래 규칙 버전·모든 입력·가격을 복원하지 못하거나 부적격인 기업을 보류했는가. 이전 종가 기준 가격을 새 방식으로 바꾸지 않았는가.
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
- 빈 섹션·미달·분할 게시·채널 검색 대체·재실행·보류 사유
- 종가·시가총액·추세 조회 실패 기업, 달력·기준일 불일치 오류·warnings·대체 출처·매매가 미산출 여부

후보·선정 근거:
- 선정 기업과 탈락 상위 후보의 저평가 근거 등급·실적 지속성 등급·신호 점수·순위·탈락 사유
- 저평가 지표·비교 대상·기간·기준일·원값/보정값·할인율·기사 발행일/URL·할인 이유 점검 결과
- 가격 규칙 버전·EPS/BPS·비교배수와 단위/기간·각 기준가치·최솟값·목표가 상한·안전마진·매수상한·최종 진입가·가격상태 및 모든 입력의 기사 근거
- 실적 대상 기간·실제 이익/현금흐름·일회성 요인·자금조달 점검 결과
- 추세 출처·심볼·기준일·80세션 여부·수정종가·두 이동평균·조건별 판정·eligible
- 인정 신호의 기사 발행일·사건일·URL과 실제치/컨센서스·기존/새 값 등
- 근거 부족으로 0점인 항목과 이유

위험 점검:
- 점검한 모든 후보의 검색어 네 묶음·기간·판정
- 제외·미완료 근거의 URL·발행일·사건 요약 또는 검색 오류

오류:
- 실패한 메시지(부모/한국/미국/실행 정보)·시도 횟수·오류, 스냅샷 저장·복원 오류, 없으면 없음
```
