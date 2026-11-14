# Daily Stock Brief

매일 오전 7시(KST)에 Claude Code 루틴이 읽는 유일한 실행 지시서다.
한국·미국 상장 기업을 시장별로 최대 1개 골라 Slack `#daily-stock-brief`(ID `C0CDFHG8PL7`)에 부모 메시지와 스레드로 바로 게시한다. 사용자가 이 결과를 실제 투자 판단에 참고하고 게시 전 사람의 검토가 없으므로, 모든 기준은 선정 수보다 정확성을 우선한다.
공시 원문·재무제표를 직접 분석하거나 미래 실적·자산을 자체 추정하지 않는다. 저평가 근거와 진입가는 제공사·기사에서 확인한 주당 실적·자산과 비교배수에 기초한 상대가치 참고값이며 수익이나 가격 하한을 보장하지 않는다.

> `CLAUDE.md`와 `README.md`는 이 파일의 심볼릭 링크다. 지침은 `AGENTS.md`만 수정한다.
> 루틴 프롬프트는 `AGENTS.md 파일을 읽고 그대로 수행해.`뿐이므로 실행에 필요한 지시는 모두 이 파일 안에 둔다.

## 독자와 범위

원문 언어와 관계없이 한국어로 쓰며, 부모 메시지는 1분, 기업 스레드는 합쳐서 2분 안에 읽을 수 있게 한다.

| 섹션 | 대상 | 최대 개수 |
|---|---|---|
| 🇰🇷 한국 | 유가증권·코스닥 상장 기업 | 1개 |
| 🇺🇸 미국 | NYSE·Nasdaq 상장 기업 | 1개 |

양국에 상장된 기업은 주 상장 거래소로 분류한다. 시장마다 `신뢰도 정상` 기업을 우선하고, 없으면 `신뢰도 낮음` 기업을 쓰며, 그것도 없으면 비워 둔다. 하루에 기억할 기업을 시장별 하나로 줄이려는 상한이다.

### 공통 판정 원칙

- 확인 불가는 통과가 아니다. 제외·보류·미조사 후보는 순위·빈자리 충원·추천 목록·가격 블록에 쓰지 않는다.
- 선정 수를 채우려고 기간·필수 조건을 완화하지 않는다. 유일한 예외는 추세 조건만 미충족한 **신뢰도 낮음 후보**다.
- 미확인 항목은 필요한 숫자·기간·정의를 넣어 최대 두 번 추가 검색한 뒤 보류한다. 기사 검색에서 공식 실적 발표 검색으로 전환한 것도 이 횟수에 포함한다. 필수 검색의 일시적 오류는 같은 검색을 한 번 재시도하고, 계속 실패하면 조회 오류로 기록한다(Slack 전송 재시도와 별개).
- 비교·할인율·가격은 Python `Decimal`로 계산하고 반올림 전 값으로 판정한다. 표시용 반올림 값(`daily_change_pct`·`upside_pct`·이동평균 표시값 등)으로 판정하지 않는다.
- 사실·수치·일정·고유명사는 검색 결과나 스크립트로 확인한 것만 쓴다. 검색 요약에 없는 본문을 읽은 것처럼 쓰거나 숫자·기간을 보완하지 않는다.

### 날짜 기준

실행 시작 시 `TZ=Asia/Seoul date +%F`로 브리핑 날짜를 한 번 확인해 고정하고 모든 스크립트에 같은 날짜를 넘긴다. 실제 실행 시각과 무관하게 **브리핑 날짜 07:00 KST**를 자료 기준 시각으로 삼고, 주말·휴장일에도 기간을 늘리지 않는다.

| 용도 | 기간·조건 |
|---|---|
| 신규 긍정 신호(신호 기간) | 직전 7일(168시간) 안에 사건 발생이 확인돼야 하며, 기사 근거는 발행일도 7일 안이어야 함 |
| 저평가·실적 지속성 확인, 목표주가·위험 점검 | 직전 30일 안에 발행된 자료 |
| 기업 중복 확인 | 오늘을 제외한 최근 7일의 KST 날짜 |
| 종가·추세 | 직전 거래일(추세는 그날까지 80거래일) |

- **발행일**은 URL의 날짜나 검색 결과에 직접 나온 발행일로만 확인한다. 기사 번호·내용·앞뒤 기사로 추정하지 않는다.
- **사건일**은 실적·리포트 발표일, 계약·공시일 등으로 확인한다. `이날 발표`처럼 발행일과의 관계가 명시돼도 된다. 발행일만으로 추정하거나 과거 사건의 재소개를 새 신호로 쓰지 않는다.
- 시각·시간대가 있으면 KST로 환산한다. 기간 경계에서 날짜만으로 포함 여부를 판단할 수 없으면 신호를 인정하지 않는다.
- 실적 자료는 기사 발행일과 실적 대상 기간을 구분한다.

## 소스와 조회 원칙

- 기사·공시는 직접 열지 않고 **WebSearch의 제목·URL·요약**으로 확인한다(루틴 환경은 WebFetch가 제한된다). 예외로 **신호 조회** 스크립트의 정형 데이터(DART 공시 목록, Yahoo 투자의견·목표가·실적 발표, 네이버 리포트 목록)는 근거로 쓴다. 공시 본문의 금액·주당 배당금과 EDGAR 공시 내용은 보도 기사로 확인한다.
- 종가·등락률·시가총액·추세는 **시세 조회**, EPS·BPS·동종/자사 과거 PER·PBR·분기 영업이익·영업현금흐름은 **가치·실적 지표 조회**의 제공사 표시값(한국 네이버 금융, 미국·분기 실적 Yahoo Finance)을 우선 쓴다. 기사 검색은 신호·할인 이유·위험 판정과 제공값이 없는 비교의 보충에 쓴다.
- 허용하는 계산은 `직전 종가 ÷ 제공 EPS·BPS`, 미국 자사 과거 PER용 `회계연도 말 종가 ÷ 제공 연간 희석 EPS`, 비교값의 중앙값, 상대 할인율, 가치기준 진입가뿐이다. 이익·자산·미래 매출을 추정하거나 재무제표 항목을 조합해 새 지표를 만들지 않는다.
- 스크립트가 주지 않는 EPS/BPS·실제 이익·현금흐름·규제자본 수치가 기사에서 부족하면 **회사 공식 실적 발표문·IR 보도자료의 검색 결과**를 보조 근거로 쓴다. 명시된 수치·정의를 옮기는 범위이며 발행일·대상 기간·기준·원문 URL을 확인하고 30일 제한을 적용한다. 이 지침의 `기사 근거`는 이 보조 근거를 포함한다. 동종·과거 배수 비교, 목표주가(기사 출처일 때)와 위험 판정은 보도 기사로 확인한다.
- 링크는 검색 결과 또는 이 지침에서 허용한 정형 데이터로 확인한 원문 URL만 쓰고 `utm_*` 등 추적 파라미터를 제거한다. Google News 리다이렉트 URL은 쓰지 않으며, 다음뉴스(`v.daum.net`)는 원문 URL을 찾지 못했을 때만 쓴다.
- 기업별 긍정 신호 대표 링크는 가장 신뢰도 높은 원문 하나다. 한국 공시 제목만으로 인정한 신호는 DART 원문 URL과 접수일을 쓴다. 별도 위험 점검 기사를 본문에 인용하면 발행일과 링크를 붙인다.
- `#daily-news-brief`는 읽지 않으며 그 채널과의 기업 중복은 제한하지 않는다.

다음 매체는 **도메인 필터에 넣지 않되**, 필터 없는 검색 결과에 나오면 링크로 쓸 수 있다: Reuters, AP, BBC, NYT, The Guardian, Politico(EU), The Verge, Ars Technica, USA Today, The Independent, El País, 연합뉴스(`yna.co.kr`). 아래 후보로 결과가 부족하면 다른 매체도 쓸 수 있다.

| 검색 후보 | 매체 |
|---|---|
| 한국 | 머니투데이(`mt.co.kr`), 파이낸셜뉴스(`fnnews.com`), 아주경제(`ajunews.com`), 이투데이(`etoday.co.kr`), 서울신문(`seoul.co.kr`), 한국일보(`hankookilbo.com`), 경향신문(`khan.co.kr`), 세계일보(`segye.com`), 노컷뉴스(`nocutnews.co.kr`), 뉴시스(`newsis.com`), 헤럴드경제(`heraldcorp.com`), SBS(`news.sbs.co.kr`) |
| 한국 증권 | 한국경제(`hankyung.com`), 매일경제(`mk.co.kr`), 서울경제(`sedaily.com`), 이데일리(`edaily.co.kr`) |
| 미국 | CNBC, Bloomberg, Yahoo News, CNN, NBC News, CBS News, ABC News, NPR |
| 미국 증권 | Yahoo Finance, MarketWatch, Barron's, Investor's Business Daily |

## 기업 선정

조사는 **긍정 신호·중복 확인 → 시세·급등·추세 → 저평가·실적 지속성 → 네 묶음 위험 점검 → 가치기준 진입가·순위** 순이다. 긍정 신호·저평가·실적 지속성·위험 점검·추세·가치기준 진입가의 여섯 조건은 모두 필수이며, 추세만 미충족한 기업은 `신뢰도 낮음`으로만 쓸 수 있다. 자사주 소각·배당 확대·목표가 상향은 조사 계기일 뿐 다른 조건을 대신하지 못한다. 제외·보류가 확정된 후보는 후속 조사를 중단하고 미수행 단계를 기록한다. 추세 미충족(`trend.eligible == false`, `trend.error` 없음)은 제외·보류가 아니므로 후속 조사를 계속한다.

### 조사 범위와 종료

- **한국 탐색**: `신호 조회 --discover-kr`로 신호 기간의 DART 자사주·배당·공급계약 공시와 네이버 종목분석 리포트 목록(`research_reports`)을 모은다. DART가 실패하면 그 세 종류를, 리포트 목록이 실패하면 목표주가·투자의견 상향을 기사로 검색한다. 리포트 목록은 후보 발견 경로일 뿐이며 상향 여부는 상세 검증에서 기사로 확인한다.
- **미국 탐색**: `신호 조회 --discover-us`로 S&P 500·S&P MidCap 400 구성 종목의 신호 기간 투자의견·목표가 상향(`target_raises`)과 Yahoo 실적 캘린더의 EPS 컨센서스 상회(`latest_earnings`)를 모은다. 구성 종목 밖의 기업은 기사 검색으로 보충한다. 구성 종목·실적 캘린더 조회가 실패하면 실패한 종류를 기사로 검색한다. `failed` 종목은 해당 신호가 미확인이다. 탐색 결과는 후보 조회 결과와 같은 근거로 쓴다.
- **기사 탐색**: 시장별로 아래 신호 여섯 종류를 각각 한 번 이상 검색한다. 스크립트 탐색에 성공한 종류(한국은 DART 세 종류와 리포트 목록이 성공하면 목표주가·투자의견 상향, 미국 두 종류)는 제외한다. 회사명 없는 검색에도 신호 기간의 시작·끝을 명시하며, 여러 종류를 한 호출에 묶어도 종류별 검색어·결과·오류를 구분한다. 한 종류의 결과로 다른 종류까지 조사했다고 기록하지 않는다.
- 식별한 기업을 시장·종목코드/티커로 합쳐 후보 목록을 저장한다. 코드가 미확인이면 시장·회사명으로 임시 식별하고 나중에 병합한다. 같은 기업의 여러 기사·신호를 후보 수에 중복 가산하지 않는다.
- **빠른 확인(한도 없음)**: 발견한 모든 후보에 중복 확인과 **시세 조회**를, 중복·급등·시세 오류가 없는 후보 전체에 **가치·실적 지표 조회**를 일괄 실행하고 저평가 1~3단계까지 마친다. 기사 검색이 필요 없으므로 조사 한도와 무관하며, 여기서 확정된 제외·조회 오류 보류는 한도에 세지 않는다. 추세 미충족, 저평가 미확인(`passed: null`), `sustainability`의 `false`·`error`는 여기서 제외하지 않는다. 실행 제한에 걸리면 나눠 실행하며, 빠른 확인을 끝내지 않은 채 후보를 `미조사`로 남기지 않는다. 그래도 끝내지 못한 후보는 `미조사: 실행 제한(빠른 확인)`으로 따로 세고 막힌 원인(도구 오류 원문·시간 초과 등)을 기록한다.
- **상세 검증(시장별 최대 12개)**: 빠른 확인을 통과한 후보의 기사 기반 검증(신호 근거·할인 이유·실적 보충·네 묶음 위험·목표가)을 다음 그룹 순서로 한다. ① 저평가 비교 통과·지속성 `true`·추세 통과 → ② 저평가 비교 통과·지속성 `true`·추세 미충족 → ③ 저평가 비교 통과·지속성 `false`/`error` → ④ 저평가 미확인. 그룹 안에서는 스크립트 기준 저평가 등급 → `sustainability.grade` → 신호 사전 근거 → 가장 최근 인정 사건일 → 종목코드/티커 오름차순이다. 신호 사전 근거는 상세 검증 전에 정형 데이터·탐색 결과로 알 수 있는 값이다. 미국은 스크립트가 인정한 신호 점수 합계(`latest_earnings` 상회 2점 + `target_raise_score`)가 높은 순이며 기사 탐색으로만 발견한 후보는 0으로 둔다. 한국은 DART 긍정 공시(`disclosures`)가 있거나 기사 탐색 결과에 신호 사실이 나온 후보 → 신호 기간 리포트(`research_reports`)의 증권사가 2곳 이상인 후보 → 1곳인 후보 순이다. 사전 근거는 조사 순서를 정하는 데 쓰며, 최종 신호 점수는 신호 인정 규칙으로 확정한다. 한국 공시 제목 신호도 후보별 조회의 후속 변경 점검을 통과해야 확정한다. 이는 조사 순서이며 최종 순위·등급은 상세 검증 후 확정한다. 신호 근거가 미완료이면 신호부터 확인하고, 불명확한 점수를 임의로 높이지 않는다. 한도를 넘긴 후보는 그룹 번호를 붙여 `미조사: 조사 한도(그룹 ④)`처럼 남긴다. 한도 안에서 시작했지만 끝내지 못한 후보는 `미조사: 실행 제한(상세 검증)`과 막힌 원인을 기록한다.
- 선행 조건을 통과한 후보는 네 묶음 위험 검색을 모두 마쳐야 선정할 수 있다. 후속 검색에서 선행 판정을 바꾸는 자료가 나오면 해당 단계로 돌아가 재검증한다. 탈락 후보의 빈자리는 남은 조사 대상에서 같은 절차로 확인하며, 선정 수를 채운 뒤에도 남은 조사 대상은 판정하거나 미조사로 명시한다.
- 독립적인 검색·조회는 도구가 지원하면 묶어서 호출하되 결과는 항목별로 확인한다. 한도나 실행 제한 때문에 조사하지 못한 기업·신호 종류를 숨기지 않는다.

### 조사 기록과 상태

선정 0개인 실행도 후보 발견부터 기록한다. 후보마다 단계별 `통과/미충족/미확인/오류/미수행`, 근거·검색어·조회 시각, 최초로 막힌 단계와 대표 사유 하나, 추가 사유, `confidence`(`normal`/`low`/`null`)를 저장한다. 미수행 단계는 통과로 간주하지 않는다.

| 후보 최종 상태 | 판정 |
|---|---|
| 선정 | `신뢰도 정상` 또는 `신뢰도 낮음` 조건을 통과했고 시장별 최종 1위 |
| 적격 미선정 | 위 두 신뢰도 중 하나를 통과했지만 순위에서 밀림 |
| 제외 | 확인된 중복·신호 요건 미충족·급등·저평가 미충족(제공 EPS 0 이하 포함)·리츠(저평가 판정 미지원)·위험 제외 사유·상승여력 미달 |
| 보류 | 신호 날짜·실제치/컨센서스·EPS/BPS·배수 정합성·현금흐름 등 필수 근거 미확인 또는 조회 오류 |
| 미조사 | 조사 한도·실행 제한 등으로 필요한 조사를 시작하거나 끝내지 못함 |

- 추세 미충족과 다른 미충족·보류 사유가 함께 있으면 최종 상태와 대표 사유는 그 다른 사유로 정하고, 추세 미충족은 추가 사유로 남긴다.
- 다른 인정 신호가 없는 후보의 신호 점수가 근거 부족으로 0점이면 보류, 확인한 사실이 요건에 맞지 않아 0점이면 제외다.
- 시장별로 `발견 = 선정 + 적격 미선정 + 제외 + 보류 + 미조사`를 코드로 확인한다. 선정·적격 미선정 수에는 그중 신뢰도 낮음 수를 함께 적는다. 단계별 진입·통과 수와 대표 사유별 수는 따로 집계하되 한 후보를 같은 집계에 두 번 세지 않는다. 미조사는 `조사 한도(그룹 ①~④)`·`실행 제한(빠른 확인)`·`실행 제한(상세 검증)`을 서로 다른 대표 사유로 세고 하나로 합쳐 표시하지 않는다. 시장 탐색 검색 자체의 실패는 후보 수가 아니라 검색 오류로 기록한다.
- 조회 오류는 스크립트가 출력한 `error`·`warnings`·`retry`의 원문(출처·예외 이름·메시지)을 그대로 저장한다. `결측`·`요청 제한`처럼 원인을 추정해 바꿔 쓰지 않는다. 대표 사유별 집계에서 조회 오류는 `출처·예외 이름`(예: `Yahoo Finance: YFRateLimitError`, `ValueError: 종가 기준일 불일치`)으로 묶어 세고 유형마다 원문 예시 하나를 남긴다. Slack의 주요 사유도 이 유형명을 쓴다.
- 상태별·대표 사유별 집계는 스냅샷의 집계 객체 하나에 저장하고, 부모의 `검증:` 줄, 빈 시장 답글의 검증·건수 줄, 실행 정보의 집계·주요 사유 줄을 모두 이 객체에서 코드로 만든다. 주요 사유는 건수 내림차순(같으면 사유명 오름차순)으로 뽑으며, 메시지마다 사유를 따로 고르거나 사유명을 바꿔 쓰지 않는다.

시장 상태는 `조사 범위 미완료`(탐색 종류 누락, 그룹 ①~③의 `조사 한도` 미조사, 또는 `실행 제한` 미조사 있음) → `조회 오류`(판정에 필요한 도구 오류 미해결) → `근거 미확인`(보류 있음) → `조사 완료` 순으로 처음 해당하는 값을 쓰고 복수 사유는 모두 보존한다. `조사 완료`는 이 실행의 검색·발견 범위에 한정하며 시장 전체를 조사했다는 뜻이 아니다. 그룹 ④(저평가 미확인)만 조사 한도로 남았다면 상태를 낮추지 않되 미조사 건수는 그대로 집계·표시한다.

### 긍정 신호와 점수

다음 중 하나 이상을 충족해야 한다. 후보마다 **신호 조회** 스크립트를 먼저 실행하고, 스크립트가 확인하지 못한 항목만 기사로 검색한다. 끝내 확인하지 못한 항목은 0점이다. 스크립트는 날짜만 있는 자료의 경계 날짜(신호 기간 시작일·브리핑 날짜)를 인정하지 않으며, `errors`·`error`는 해당 근거의 조회 오류로 기사 검색으로 보충한다.

| 스크립트 근거 | 인정 방법 |
|---|---|
| 미국 `latest_earnings` | `in_signal_window`·`beat`가 모두 `true`이면 **실적 컨센서스 상회 2점**. `basis`의 Yahoo 제공 EPS로 추정·실제가 같은 제공사 기준이며 발표 시각이 사건일이다. |
| 미국 `target_raises` | 신호 기간 안의 투자의견·목표가 상향(최초 커버리지 제외). **`target_raise_score`**를 점수로 쓰며 증권사·시각·기존/새 목표가를 기록한다. |
| 한국 `disclosures` | 후보별 조회의 `title_only_eligible == true`인 신규 자사주 취득·소각 결정·공급계약 체결 공시는 기사 없이 해당 항목 1점을 인정한다. DART 접수일이 사건일이며 금액·주식 수·계약 상대방·기간은 확인된 경우만 쓴다. 탐색 결과만으로 확정하지 않으며 아래 한국 공시 검증 규칙을 적용한다. 배당은 기존·새 주당 배당금과 확대 결정을 기사로 확인해야 한다. |
| 한국 `research_reports` | 신호 기간 리포트의 증권사·날짜 목록이다. 목표가·투자의견 상향 여부는 기사로 확인하며 목록만으로 점수를 주지 않는다. |
| 한국 `earnings_releases` | 잠정실적 공시일(사건일) 확인용이다. 컨센서스 상회는 기사로 확인한다. |

| 신호 | 점수 | 인정 조건 |
|---|---|---|
| 실적 컨센서스 상회 | 2 | 매출·영업이익·순이익·EPS 중 지표를 밝히고 같은 기간·통화·회계 기준의 실제치와 컨센서스를 확인. 전년 대비 증가만으로 인정하지 않으며 GAAP·조정 수치를 섞지 않음 |
| 가이던스 상향 | 2 | 회사가 상향을 발표하고 동일 지표·기간의 기존 값과 새 값을 확인. 범위는 양쪽 모두 확인하며 상단 하나로 추정하지 않음. 최초 제시·유지·애널리스트 전망 상향은 제외 |
| 목표주가·투자의견 상향 | 증권사 1곳 1점, 2곳 이상 2점 | 증권사·리포트 발표일·상향 사실 확인. 유지·최초 커버리지·과거 상향 재소개는 제외 |
| 수주·공급계약 공시 | 1 | 한국의 제목 검증 통과 공시는 신규 체결 사실만으로 인정하며 계약 대상의 기사 확인은 생략한다. 그 밖에는 계약 대상과 체결·수주 공시를 확인한다. MOU·협상·기대는 제외하며 상대방·금액·기간은 확인된 경우만 사용 |
| 자사주 매입·소각 | 1 | 한국의 제목 검증 통과 공시는 신규 취득·소각 또는 취득 신탁계약 체결 결정 사실만으로 인정하며 금액·주식 수 확인은 생략한다. 그 밖에는 신규 결정 또는 실제 이행 공시와 금액 또는 주식 수를 확인한다. 결정·이행·신탁계약을 구분하고 계획 재소개·의향 표명은 제외 |
| 배당 확대 | 1 | 같은 기준의 기존·새 주당 배당금과 확대 결정 공시 확인. 유지·검토·주가 하락에 따른 배당수익률 상승은 제외 |

각 항목은 한 번만 센다. 같은 실적의 여러 기사나 같은 자사주 계획의 결정·이행을 중복 가산하지 않는다. 주가 상승만으로는 긍정 신호를 인정하지 않는다.

**한국 공시 제목만으로 인정하는 범위**

- 후보별 DART 조회가 완전하고, 해당 회사의 공시가 신호 기간 안에 있으며, `title_only_eligible == true`일 때만 기사 확인을 생략한다. 스크립트의 정확한 신규 제목 목록에 해당하는 자사주 취득·소각 결정·취득 신탁계약 체결 결정·단일판매 공급계약 체결에 한정한다. 공시 종류별 점수는 최대 1점이며 여러 계약·결정이 있어도 가산하지 않는다.
- 정정·변경·연장·추가·재공시, 해지·철회·취소·해제, 자회사 공시와 애매한 제목은 자동 인정하지 않는다. 후보별 조회는 최근 30일 목록에서 같은 신호 종류의 후속 변경도 보존한다. 원공시 당일부터 기준 시각까지 후속 변경 가능성이 있으면 같은 사건인지 제목만으로 연결하지 않고 기사 확인 대상으로 돌린다. 기준일 당일처럼 시각 미확인인 후속 변경도 해결 전에는 자동 인정하지 않는다.
- 기사로 보충할 때는 원공시의 사건일·변경과의 관계·현재 유효한 결정/체결 사실을 확인한다. 정정일·연장일을 신규 사건일로 쓰지 않으며, 신호 기간 밖 원공시의 정정·변경만으로 점수를 주지 않는다. 철회·해지된 사건은 인정하지 않고, 원공시·변경 관계가 끝내 미확인이면 해당 신호는 0점이다. 다른 인정 신호가 없으면 근거 부족은 보류, 확인된 요건 미충족은 제외다.
- 제목으로 인정한 경우 `공시 제목 확인`과 원문 URL·접수일·확인 방식·생략한 세부 항목을 기록한다. 금액·주식 수·상대방·기간을 채우기 위한 추가 기사 검색은 필수가 아니다. 배당 확대·목표가/투자의견 상향·실적 상회·가이던스 상향의 비교 근거와 저평가·실적 지속성·위험 점검은 기존대로 확인한다.

### 저평가 근거 — 필수

**가치·실적 지표 조회**의 `valuation.comparisons`를 쓴다. 자사 배수는 직전 거래일 종가 ÷ 제공 EPS·BPS(실제 최근 4분기 합산 또는 최근 분기)이고, 비교 기준은 제공사 표시값의 중앙값이다.

| 비교 | 한국 | 미국 | 최소 표본 |
|---|---|---|---|
| 동종 PER·PBR | 네이버 동일업종 비교 종목의 PER·PBR | Yahoo 업종 상위 기업(최대 8개)의 trailing PER·PBR | 양수 3개 |
| 자사 과거 PER | 네이버 최근 3개 실제 회계연도 PER | Yahoo 최근 3개 회계연도마다 `결산일 이전 마지막 종가 ÷ 연간 희석 EPS`(둘 다 분할 반영 값) | 양수 3개 |

1. **주 지표 선택**: 일반 기업은 PER 비교를, 금융·보험은 PBR 비교와 PER 비교를 함께 쓴다. 리츠는 감가상각 때문에 PER이 가치를 나타내지 못하고 이 루틴의 데이터로 FFO 배수 비교를 할 수 없으므로, 제공사 업종 분류나 회사 소개로 리츠임을 확인하면 `제외: 리츠(저평가 판정 미지원)`로 둔다. 일반 기업의 PBR 비교는 참고로만 기록한다. 제공 EPS(`eps.value`)가 0 이하로 확인되면 `제외: 저평가 미충족(EPS 0 이하로 PER 근거 없음)`이다(일반·금융 공통). EPS 값 자체가 없거나 조회에 실패한 경우만 미확인으로 보류한다.
2. **판정**: 주 지표로 확인한 비교(`passed`가 `true`/`false`)가 모두 `true`여야 한다. 하나라도 `false`면 저평가 미충족으로 **제외**한다. 표본 부족(`passed: null`)은 미확인이다.
3. **비교 가능성 점검**: 동종 목록은 제공사 분류라 사업 구성이 다를 수 있다. 스크립트는 할인율 50% 이상인 비교를 `passed: null`로 돌린다. 그 밖에도 동종 종목명과 배수를 보고 사업이 명백히 다른 종목(지주사·적자 회복기 등)이 중앙값을 좌우하면 해당 비교를 미확인으로 돌린다. 자사 과거 PER은 스크립트가 표본 연도의 연간 EPS를 최근 4분기 EPS와 비교해, 절반 미만인 연도(이익 회복기·급증으로 과거 배수가 부풀려짐)가 있거나 연간 EPS를 확인하지 못하면 `passed: null`로 돌린다. 일부 종목·연도만 빼서 다시 계산하거나 50% 기준을 바꾸지 않는다.
4. **할인 이유 점검**: `회사명 일회성 OR 처분이익 OR 평가이익 OR 환급 OR 법인세 효과` / `티커 one-time gain OR one-time benefit OR non-recurring OR tax benefit OR refund OR impairment`로 30일 기사·공식 발표를 검색한다. 최근 4분기 이익에 일회성 이익·환급·세금 효과가 포함됐다는 기사·공식 발표가 있으면 금액·영향의 확인 여부와 관계없이 PER 비교를 미확인으로 돌리고, 그 효과를 뺀 정상 이익 기준의 기사 비교가 없으면 **보류**한다. 이 사실을 리스크 줄로 설명해 통과시키지 않는다. PBR을 쓰면 최근 회계연도 ROE 양수(한국 `history.roe`, 미국 `roe`)와 자산 부실 기사 미발견을 확인한다.
5. **기사 비교 보충(선택)**: 스크립트로 확인하지 못한 비교는 `회사명 밸류에이션 동종업계 PER PBR` / `티커 valuation peers historical multiple`로 30일 기사에서 보충할 수 있다. 지표 정의·실제/예상·대상 기간·회계 기준·평가 기준일을 확인하고, 직전 종가 기준이 아니면 기준주가와 EPS·BPS가 함께 있고 이후 실적·주식 수 변경이 없을 때만 `기사 배수 × 직전 종가 ÷ 기사 기준주가`로 보정해 원래 값·보정값·전제를 기록한다. 단순한 `저평가` 표현·목표가 괴리·고점 대비 하락률은 인정하지 않는다.

상대 할인율은 `1 − 자사 배수 ÷ 비교 배수`다. 서로 다른 업종·지표의 할인율 크기로 순위를 매기지 않으며 `PER 10배 이하` 같은 공통 절대 기준을 쓰지 않는다.

- **등급 2**: 동종 비교와 자사 과거 비교를 모두 확인해 통과. **등급 1**: 둘 중 하나만 확인해 통과.
- **보류**: 주 지표 비교를 하나도 확인하지 못했거나(EPS 0 이하는 위 1번에 따라 제외) 할인 이유 점검이 미완료다. 스크립트 `error`는 조회 오류 보류다.

### 가치기준 진입가 — 필수

저평가 판정에 쓴 주 지표 비교마다 주당 기준가치를 계산한다. 스크립트가 통과 비교마다 출력하는 `valuation_basis`(EPS/BPS·비교 중앙값·`evidence`)를 그대로 `levels()`에 넘기고, 기사로 보충한 비교도 같은 구조로 넣는다. 확인한 비교 중 일부만 골라 넣지 않는다.

| 방식 | 입력 | 주당 기준가치 |
|---|---|---|
| PER | 제공 EPS(실제 최근 4분기 합산) 또는 기사 EPS, 동종/자사 과거 PER 중앙값 | `EPS × 비교 PER` |
| PBR | 제공 BPS 또는 기사 BPS, 동종 PBR 중앙값, ROE·자산 건전성 근거 | `BPS × 비교 PBR` |

- EPS·BPS는 제공사·기사에 표시된 값만 쓰며 주가·배수·목표가에서 역산하지 않는다. 실제/예상·대상 기간·통화·보통주/ADR 단위가 비교 기준과 맞아야 한다. 연간/TTM 배수에 분기 EPS를 곱하거나 자체 연율화하지 않으며, 추정 EPS(`cnsEps`)는 쓰지 않는다. 두 자료의 기준이 다르면 문자열만 같게 바꾸지 않고 보류한다.
- 비교배수가 범위면 하단을 쓴다. 모든 기준가치 중 최솟값을 채택하고, 목표가를 확인했으면 그 값과 비교해 더 낮은 값을 **보수 기준가치**로 쓴다. 목표가만으로 기준가치를 만들지 않는다.
- **가치기준 매수상한 = 보수 기준가치 × 0.80**, **최종 진입가 = min(매수상한, 직전 종가)**다. 20%는 이 루틴의 고정 안전마진이며 검증된 최적값이나 손실 방지 장치가 아니다. 근거가 부족한 기업을 마진만 늘려 통과시키지 않는다.
- 종가가 매수상한 이하이면 `가격조건 충족`, 초과하면 `진입대기`다. 브리핑 기준 상태이며 실시간 매수 신호가 아니다. 이후 실적·주식 수·추정치·위험·추세 근거가 바뀌면 기존 가격은 폐기한다.
- EPS·BPS 또는 비교배수가 없으면 **진입가 산출 불가로 보류**한다. 목표가 할인·종가·차트 값으로 대신하지 않는다.

### 실적 지속성 — 필수

최신 결산기의 **실제 이익과 현금 창출 수치**를 확인한다. 실현되지 않은 계약 총액·매출 전망·목표가 상향·주주환원만으로 통과시키지 않는다.

- 일반 기업은 `sustainability`를 쓴다. Yahoo 분기 재무의 최신 분기에서 **영업이익 양수와 영업현금흐름 양수**를 모두 확인해야 한다(`eligible`). 4분기 전(전년 동기)과 비교한 `yoy`를 함께 본다. 최신 분기가 200일보다 오래됐거나 항목이 없으면 `error`다.
- 30일 기사·공식 발표에 제공값보다 **새로운 결산기 실적**이 있으면 그 수치를 우선한다. 같은 기간의 영업이익과 영업현금흐름(또는 정의가 명시된 FCF)을 모두 확인해야 한다. 누적으로만 명시됐으면 `현금흐름 검증: 누적`을 표시하고 최신 분기 영업이익 양수를 따로 확인한다. 분기·누적·연간 수치를 섞거나 차감해 분기 수치를 만들지 않는다. 새 결산기의 현금흐름을 확인하지 못하면 이전 분기 제공값으로 통과시키지 않고 보류한다.
- `eligible == false`(최신 분기 영업적자 또는 영업현금 유출)는 원인과 일회성 여부를 기사로 확인한다. 일회성이 아니거나 원인을 확인하지 못하면 제외한다.
- 금융·보험은 `sustainability`를 쓰지 않고 최신 결산기 실제 순이익 양수와 적용 규제자본비율의 최소 기준 충족을 30일 기사·공식 발표로 확인한다. 확인하지 못하면 보류한다.
- 일회성 처분이익·평가이익이나 운전자본의 일시적 회수에 의존한 흑자·현금흐름이라는 기사가 있으면 반복 가능한 수익·현금 창출 근거를 추가 확인한다.
- 대규모 투자(데이터센터·공장 등)가 가치평가의 핵심이면 투자금, 확정/미확정 자금, 조달 방식(차입·임대·증자)과 비용을 확인한다. 핵심 자금조달이 미확정이거나 그 비용을 반영한 가치평가 근거가 없으면 보류한다. `증자 계획 없음`을 자금 확보로 해석하지 않는다.
- **등급 2**: 위 조건을 통과하고 전년 동기 대비 영업이익과 영업현금흐름이 모두 유지·개선(`yoy`의 두 값 `true`)됐으며 최근 4분기 중 영업현금 유출 분기가 없다(`ocf_negative_quarters_4q == 0`, `null`이면 불가). 기사 수치로 판정할 때도 4분기 유출 없음을 확인해야 한다. 업종 대체 지표는 이익·건전성이 모두 유지·개선돼야 한다.
- **등급 1**: 위 조건은 통과했지만 전년 동기 비교가 없거나, 4분기 중 유출 분기가 있거나(리스크 줄에 분기와 금액을 밝힌다), 악화 항목의 원인과 반복 가능성을 기사로 확인한 경우다. `yoy` 악화의 원인을 확인하지 못하면 보류한다.

### 하락 추세 완화 — 필수

**시세 조회**의 `trend.eligible == true`인 기업만 `신뢰도 정상`이 될 수 있다. 분할·배당을 반영한 동일 수정주가 계열의 연속 80세션으로 다음 두 조건을 모두 확인한다.

- 직전 거래일 **수정종가 ≥ 최근 60거래일 수정종가 이동평균**
- 최근 **60거래일 이동평균 ≥ 20거래일 전의 60거래일 이동평균**

`false`는 추세 미충족으로 **신뢰도 낮음 후보**로만 쓸 수 있다. `error`·누락은 보류이며 신뢰도 낮음 후보도 될 수 없다. 종가 한두 번의 반등, `바닥`·`반등 기대` 같은 기사 표현, 기사 종가로 추세를 대신하지 않는다. 60일·20일은 운영 기준이며 검증된 최적값이 아니다. 반등 초기를 놓칠 수 있고 통과 후에도 하락할 수 있으며, 추세는 기업가치의 증거가 아니다.

### 신뢰도 낮음 후보

시장별 `신뢰도 정상` 기업이 없을 때만 그 자리를 채운다.

- **대상**: `trend.eligible == false`이고 `trend.error`가 없으며, 추세 이외의 필수 조건(중복·신호·급등, 저평가 등급 1 이상, 실적 지속성 등급 1 이상, 위험 점검의 제외·보류 사유 미발견, 가치기준 진입가 산출, 목표가 확인 시 상승여력 15% 이상)을 모두 통과한 후보. 검증 기준·검색 횟수는 일반 후보와 같다.
- **순위**: 모든 `신뢰도 정상` 기업 뒤에 두고, 서로는 **상승여력과 선정 순서**의 비교 순서를 쓴다.
- **표시**: 부모·답글에 `⚠️ 신뢰도 낮음`과 미충족 조건을 함께 밝힌다. 가격은 `levels()`로 같게 계산해 스냅샷에 저장하되, 가격 블록에는 수익실현·손절 줄 대신 `• 수익실현·손절: 추세 조건 통과 전 미산출`을 쓴다.
- **집계**: 최종 상태는 `선정` 또는 `적격 미선정`이며 `confidence: "low"`와 `trend`의 두 판정을 저장한다. 게시된 기업은 다음 실행의 7일 중복 제외 대상이다.

### 제외 조건과 위험 점검

시세 조회의 `surge_excluded`가 `true`이거나, 신호 기간 기사에서 한국 하루 15% 이상·미국 20% 이상 상승 또는 상한가가 확인되면 제외한다.

선행 조건을 통과한 후보마다 회사명(가능하면 종목코드·티커)과 30일 기간을 지정해 네 묶음을 각각 검색한다. 한국은 한국어, 미국은 영어로 검색한다. 한국은 `risk_disclosures`(30일 DART 희석·소송·규제 공시)도 희석·규제·소송 묶음의 근거로 판정하며, 조회 실패 시 기사만으로 판정했음을 기록한다. 미국 `target_cuts_30d`는 제외 사유가 아니라 리스크 줄과 추정치 하향 점검의 단서다. 선행 제외·보류로 건너뛰면 네 항목을 `미수행: 선행 단계와 사유`로 남긴다.

| 묶음 | 한국어 | 영어 |
|---|---|---|
| 실적·전망 | `실적 하회 OR 컨센서스 하회 OR 가이던스 하향` | `earnings miss OR revenue miss OR guidance cut OR lowered outlook` |
| 희석·규제·소송 | `유상증자 OR 전환사채 OR 규제 OR 소송` | `share offering OR convertible notes OR regulatory action OR lawsuit` |
| 이익 추정치 | `이익 추정치 하향 OR EPS 하향 OR 실적 전망 하향` | `earnings estimate cut OR EPS revision down OR earnings forecast lowered` |
| 현금·자금조달 | `영업현금흐름 악화 OR 자금조달 OR 차입 부담` | `negative operating cash flow OR financing risk OR funding gap` |

- **실적·희석 제외**: 30일 기사·DART 공시에서 해당 기업의 실적 하회·가이던스 하향·희석 공시를 확인한 경우 제외한다.
- **소송·규제 판정(양 시장 공통)**: 제목·키워드만으로 제외하지 않고 당사자·대상·청구 또는 조치 내용·진행 단계·기사/공시에 명시된 영향을 기록한다. 회사가 피고라는 사실이나 조사 개시만으로 확정된 손실·제재로 표현하지 않는다.
  - **제외**: 회사의 배상책임을 인정한 불리한 판결·결정, 핵심 영업/자산을 제한하는 가처분, 실제 영업정지·허가취소·주요 사업 제한·회계부정 제재·관리종목 지정·매매거래정지 등을 확인한 경우. 영업·자산의 핵심성이나 사업 영향은 기사·공시에 명시된 근거로 확인하며 자체 금액 추정이나 임의 중요도 기준을 만들지 않는다.
  - **보류**: 회사에 대한 불리한 청구·진행 중 조사 등이 있으나 책임·진행 단계·사업 영향을 확인하지 못해 제외 여부를 판정할 수 없는 경우. 필요한 근거를 최대 두 번 추가 검색한 뒤 미확인으로 남긴다.
  - **해당 사유로 제외·보류하지 않음**: 회사가 원고인 권리 행사, 회사에 유리한 기각·종결, 해당 회사에 적용되지 않는 업종 제도 설명, 일반적 가능성인 것으로 확인되고 별도의 불리한 영향 근거가 없는 경우. 미해결 불리한 사건을 유리한 사건 하나로 상쇄하지 않는다. 위험 항목이 있다는 이유로 다른 필수 조건을 생략하지 않는다.
- **추정치 하향은 보류**: 증권사·발표일·동일 지표와 기간의 기존·새 값을 확인한다. 하향 이후 같은 기준의 추정치 유지·상향과 하향된 이익을 반영한 새 저평가 근거를 모두 확인한 경우에만 재검토한다. 주가가 더 많이 하락했다는 이유로 통과시키지 않는다.
- **현금·자금조달**: 실제 지속적인 현금 유출·구조적 악화는 실적 지속성 규칙으로 제외하고, 핵심 자금조달 미확정·비용 미반영은 보류한다. 키워드만으로 판정하거나 계약 총액을 순이익·기업가치로 대체하지 않는다.
- **위험 점검 미완료**: 악재의 발행일·대상·사건을 확인하지 못하거나 필수 검색이 실패하면 확정하지 않고 다른 후보를 계속 점검한다.
- 네 검색을 정상 완료했으나 근거를 찾지 못하면 `해당 검색에서 제외·보류 사유 미발견`이다. `위험 없음`·`안전`으로 표현하지 않으며, 실적 지속성의 긍정적 수치 확인과도 별개다.

### 상승여력과 선정 순서

상승여력은 `목표주가 ÷ 직전 종가 − 1`인 보조 지표이며 저평가 판정이 아니다. 목표가를 확인했으면 `levels()`의 `upside_eligible`이 `false`(15% 미만)일 때 제외한다. 목표가가 미확인이어도 필수 여섯 조건을 통과하면 같은 후보군에서 비교한다. 상승여력 가점·`총점`은 쓰지 않고 후보마다 저평가 등급·실적 지속성 등급·신호 점수를 따로 기록한다.

시장별로 **신뢰도 정상 → 신뢰도 낮음** 순으로 나누고, 같은 신뢰도 안에서 **저평가 등급 → 실적 지속성 등급 → 가격조건 충족 우선 → 신호 점수 → 상승여력 원값** 순으로 비교한다. 마지막 비교에서는 목표가 확인 후보를 우선하고 `Decimal(upside_raw)`를 비교하며, 그래도 같으면 종목코드/티커 오름차순이다. 보도량·매체 수는 점수·순위·동점 처리에 쓰지 않는다. 시가총액은 규모 참고용이며 점수·동점 처리·제외·저평가 판단에 쓰지 않는다.

## 중복 방지와 재실행

채널의 `📈 Daily Stock Brief` 제목과 날짜로 부모 메시지를 식별한다(굵게 표기 유무 무시).

- 최근 7일 부모 메시지 전부의 기업을 제외한다. 스레드와 사람이 쓴 다른 메시지는 읽지 않는다. 오늘 안에서도 같은 기업을 두 번 쓰지 않는다.
- **오늘 부모가 있으면 새로 만들지 않고 그 스레드만 읽는다.** 한국·미국 섹션과 실행 정보 답글이 모두 있으면(분할 답글 포함) 아무것도 게시하지 않고 종료한다. 이미 게시된 메시지는 수정·재게시하지 않으며 새 형식을 덧붙이지 않는다.
- 빠진 답글은 부모에 적힌 기업·순서로만 쓰고, 이미 게시된 분할 항목은 건너뛴다. 채널·날짜·부모 원문이 일치하는 스냅샷에서 기업별 저평가·실적 지속성·추세·`confidence`와 가격 입력(가격 규칙 버전, `levels()`의 모든 입력, 목표가의 값/미확인 상태)을 복원하고, 네 묶음 위험 점검은 다시 한다. 복원한 입력으로 같은 규칙 버전의 `levels()`를 다시 계산해 저장된 가격·상태와 부모 상승여력에 대조한다. 가격 기간은 부모 날짜 + 28일이다.
- 스냅샷이 없으면 원래 세션 기록이나 이미 게시된 답글에서 브리핑 기준 시각 이전의 모든 입력·판정을 확인한 경우에만 복원한다. 새 시세·목표가·EPS/BPS·비교배수로 대체하거나 부모의 반올림된 상승여력에서 역산하지 않으며, 원래 기록에 없는 근거를 새 조사로 보완하지 않는다.
- 복원하지 못했거나(규칙 버전이 `value-entry-v1`이 아닌 이전 방식 포함), 재계산이 저장값과 다르거나, 위험 재점검·현재 필수 조건을 통과하지 못하면 해당 위치에 `회사명 — 보류: 사유`만 쓰고 대체 기업·가격 블록은 넣지 않는다(일반 항목·빈 섹션 형식의 예외). 부모에 `신뢰도 낮음`으로 적힌 기업은 신뢰도 낮음 후보 조건으로 확인한다.
- 빠진 빈 시장 답글의 원래 진단을 복원하지 못하면 `검증: 원래 조사 기록 복원 불가`로 쓰고 건수를 추정하지 않는다.

### 게시 전 스냅샷

상태 디렉터리는 실행 환경의 `BRIEF_STATE_DIR`, 없으면 작업 폴더의 `.brief-state`다. 로컬 Mac의 경로가 클라우드에서도 같다고 가정하지 않는다. 시작 시 절대 경로·쓰기 가능 여부·기존 파일을 확인하고, 공유·영속 저장소 설정이 없으면 `실행 간 보존 미확인`으로 기록한다. 외부 저장소·새 연동을 임의로 만들지 않는다.

- **잠금**: 신규 실행은 채널·날짜별 잠금 디렉터리(예: `C0CDFHG8PL7-YYYY-MM-DD.lock`)를 원자적으로 만들고 소유 실행 ID를 기록한다. 이미 있으면 소유 실행의 종료·게시 여부를 확인하기 전까지 게시하지 않으며, 시간 경과만으로 지우지 않는다. 종료 시 자신이 만든 잠금만 해제한다. 서로 다른 클라우드 작업 폴더 사이의 잠금은 보장되지 않으므로 게시 직전 채널을 다시 읽는 확인을 반드시 한다. 루틴을 중복 등록하지 않는다.
- **조사 기록**: `runs/YYYY-MM-DD/실행ID.json`에 후보·검색·오류를 단계마다 저장한다. 선정 0개·게시 실패·재실행 기록도 보존한다.
- **게시용 스냅샷**: 부모를 보내기 전에 `YYYY-MM-DD.json`(UTF-8, `schema_version: brief-state-v2`)을 저장한다. 가격·비율은 소수 문자열, 날짜는 `YYYY-MM-DD`, 시각은 시간대가 있는 ISO 8601(초 포함)이다. 같은 디렉터리의 임시 파일에 쓴 뒤 `os.replace`로 교체하고 다시 읽어 내용·집계·메시지 길이를 확인한다. 저장 실패 시 부모를 게시하지 않는다. 기존 부모에 연결된 스냅샷은 새 조사로 덮어쓰지 않는다.

| 저장 항목 | 내용 |
|---|---|
| 실행 식별 | 실행 ID·세션 ID/기록 경로(미확인 포함)·작업 경로·Git 커밋·AGENTS.md SHA-256·상태 경로와 보존 확인 여부·브리핑 날짜·기준 시각·예정 시각/미확인·실제 시작/각 게시/집계 시각·채널 ID·부모 원문·`parent_ts`(게시 전 `null`) |
| 조사 진단 | 시장별 여섯 신호 탐색의 검색어·기간·결과·오류, `--discover-us`의 구성 종목 수·실적 캘린더 건수·`failed`, 모든 후보의 식별자·단계별 기록·최종 상태·대표/추가 사유·조사 순서·미수행 항목·`confidence`와 신뢰도 낮음 채택 이유, 상태별/단계별/대표 사유별 집계·시장 상태, 조회 오류 유형별 건수·원문 예시·재조회 결과 |
| 기업별 근거 | 시장·종목코드/티커·회사명·순서, 신호 조회·시세 조회·가치·실적 지표 조회 결과 전체, 종가의 날짜·출처, 목표가의 모든 확인값·채택 이유·미확인 상태, 기사 URL·발행일 |
| 판정 근거 | 저평가 비교의 지표·대상·기간·기준일·원값/보정값·할인 이유·등급, 실적/현금흐름의 대상 기간·실제 수치·일회성/자금조달 점검·등급, `trend` 전체, 네 묶음 위험 검색어·기간·판정·근거와 소송/규제의 당사자·내용·진행 단계·영향, 신호별 확인 방식(공시 제목/정형 수치/기사)·원공시/후속 변경 확인·기사 생략 항목·점수·순위 |
| 진입가 근거 | `levels()` 입력(`valuation_bases` 전체)과 결과(가격 규칙 버전·각 기준가치·보수 기준가치·목표가 상한 반영·매수상한·진입가·상태·손절·수익실현) |
| 게시 내용 | 한국·미국 답글 원문(분할 순서 포함)·실행 정보용 확정 진단·메시지별 게시 여부·`ts`·시도 횟수·전송 오류 |

- 부모·답글 전송이 확인되면 `parent_ts`와 게시 결과를 바로 갱신한다.
- 재실행은 채널·날짜·부모 원문이 일치하는 스냅샷만 쓴다. 도구가 바꾼 Markdown 표기는 정규화하되 회사명·순서·수치는 그대로 대조한다. `parent_ts`가 있으면 실제 부모의 `ts`와 일치해야 하고, `null`이면 대조한 부모의 `ts`를 연결한다. 저장된 답글을 검토 없이 재전송하지 않는다.
- 상태·조사 기록·잠금 파일은 Git에 추가·커밋·푸시하지 않는다. 필요하면 로컬 `.git/info/exclude`에 상태 경로를 넣는다.

## Slack 작성

부모 1개와 한국·미국·실행 정보 답글 순으로 게시한다.

- 메시지는 URL 포함 **3,000자 이하**인지 코드로 센다. 줄여도 넘으면 완전한 기업 항목 사이에서만 나누고 제목에 이어지는 답글임을 표시한다. 추세 줄과 가격 블록은 해당 기업 항목과 함께 둔다.
- 표준 Markdown을 쓴다: 굵게 `**텍스트**`, 기울임 `_텍스트_`, 링크 `[텍스트](URL)`.
- 회사명은 한국은 한글, 미국은 영문으로 쓰며 부모·스레드의 표기·순서를 맞춘다.
- 주가·등락률에는 종가·장중·시간외 여부와 기준일을 붙이고 서로 다른 기준을 섞지 않는다.
- 매매 권유와 주가·실적 예측은 쓰지 않는다. 전망은 `회사는 …로 제시했다`, `A증권은 …로 올렸다`처럼 주체를 밝힌 인용만 허용한다. 가격 움직임만으로 펀더멘털을 판단하지 않는다.
- 기업이 없는 섹션은 부모·답글 모두 헤더 아래에 `_이번 조사에서 선정한 기업 없음_`과 `검증: 시장 상태 · 주요 사유 최대 2개(건수)`를 쓴다. 발견 0개이면 탐색 검색의 정상 완료 여부를 밝힌다. 시장 전체에 적격 기업이 없다는 뜻으로 쓰지 않는다.

### 부모 메시지

```
📈 **Daily Stock Brief** | 2026-11-02 (월)
_수집 기간: 10/26 07:00 ~ 11/02 07:00 KST_

**🇰🇷 한국**
• 회사명 — 선정 이유 한 줄 · 상승여력 26% · 진입대기

**🇺🇸 미국**
• 회사명 — 선정 이유 한 줄 · 상승여력 미확인 · 진입대기 · ⚠️ 신뢰도 낮음(미충족: 하락 추세 완화)

🧵 투자 포인트와 매매가는 스레드에서 확인 · 진입은 가치기준 매수상한·종가 중 낮은 값
```

- 날짜는 KST의 `YYYY-MM-DD (한글 요일 한 글자)`다.
- 선정 이유는 최근 신호와 정량적인 저평가 근거를 한 줄로 줄인다(예: `자사주 소각 결정·PER 7배(동종 10배)`). 비교 출처는 답글에 붙인다.
- 상승여력은 답글 목표주가 줄의 `upside_pct`(직전 종가 기준)와 같아야 하며, 목표가 미확인은 `상승여력 미확인`이다. 줄 끝에 `가격조건 충족` 또는 `진입대기`를 붙인다.
- 신뢰도 낮음 기업은 가격상태 뒤에 `⚠️ 신뢰도 낮음(미충족: 하락 추세 완화)`를 붙인다.
- 선정 기업이 있어도 시장 상태가 `조사 완료`가 아니면 기업 목록 아래에 `검증: 상태 · 주요 사유(건수)`를 붙인다. 양국 모두 선정 0개이면 마지막 안내는 `🧵 조사 범위·보류 사유와 실행 정보는 스레드에서 확인`이다.

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
• 가치기준 매수상한 64,000원 (최근 4분기 EPS 8,000원 × 동종 PER 중앙값 10배 = 80,000원 · 안전마진 20% · [네이버 금융](https://finance.naver.com/item/main.naver?code=005930) 11/2 조회)
• 수익실현 74,200원 (+16%)
• 진입 64,000원 이하 (진입대기)
• 손절 58,900원 (−8%)
🔗 [매체명](https://원문링크)
```

- **제목**: 한국은 6자리 종목코드, 미국은 티커를 괄호에 넣는다. 확인하지 못하면 괄호를 생략하고 시가총액은 미확인으로 쓴다.
- **설명 1줄**: 무엇을 만들거나 파는 회사인지 한 문장 뒤에 `market_cap.display`를 그대로 붙인다. 조회 시각을 종가 기준일로 해석하지 않으며, 실패 시 기사·주식 수 계산으로 보충하지 않는다.
- **투자 포인트는 정확히 3줄**, 각 한 문장이다.
  1. **사건**: 인정된 신호의 사건일과 확인된 수치. 실적은 지표·기간·실제치·컨센서스, 가이던스는 기존·새 값, 자사주는 결정·이행·신탁계약 구분을 밝힌다. 한국 공시 제목만으로 인정했으면 `공시 제목 확인`과 신규 결정/체결 사실·접수일·DART 링크를 쓰고 미확인 금액·주식 수·상대방·기간은 생략한다. 실제 매입 완료·매출/이익 기여를 추정하지 않는다.
  2. **저평가·실적 지속성**: 평가배수와 비교 기준(`동종 n개 중앙값`처럼 표본 수 포함), 실적·현금 창출의 실제 수치를 지표·대상 기간과 함께 쓴다. 제공값이면 `[네이버 금융](urls.display)`·`[Yahoo Finance](urls.display)` 링크와 조회일, 기사면 발행일·링크를 붙이고 보정값이면 표시한다.
  3. **리스크·다음 확인**: 기사에 나온 약점·반론을 우선하고, 없으면 무엇이 확인될 때 판단이 달라지는지 조건으로 쓴다. 확인 일정은 기사에 있는 것만 쓴다. 제외·보류 사유를 이 줄로 설명해 통과시키지 않는다.
- **추세 줄**: `trend`의 두 조건·출처·기준일을 표시한다. 향후 상승을 예측하지 않는다.

신뢰도 낮음 기업은 같은 형식에서 다음 세 곳만 다르다. 추세 줄의 값은 `adjusted_close`·`ma60`·`ma60_20_sessions_ago`를 종가 단위로 반올림하고 조건별 판정에 맞게 `≥`·`<`를 쓴다.

```
**1. 회사명 (005930) · ⚠️ 신뢰도 낮음**
📉 추세 미충족: 수정종가 a < 60일 평균 b · 60일 평균 b ≥ 20거래일 전 평균 c (Yahoo Finance, M/D 기준)
• 수익실현·손절: 추세 조건 통과 전 미산출   ← 수익실현·손절 두 줄 대신
```

빈 시장 답글은 헤더·빈 섹션 문구·`검증: 시장 상태`·`발견 n · 선정 n · 적격 미선정 n · 제외 n · 보류 n · 미조사 n`·`주요 사유: 최대 2개(건수)`만 각각 별도 줄로 쓰며, 검증·건수·주요 사유는 집계 객체에서 만든 실행 정보의 값과 같아야 한다. 빈 답글도 게시·재실행 완료 판정에서 해당 시장의 답글로 센다.

### 가격 블록

제목은 `💵 가격 (기준일 종가 금액, 출처 · 기간 M/D까지)`이며 기준일·출처·종가는 시세의 `date`·`source`·`close`, 기간은 브리핑 날짜 + 28일(이후 이 참고값을 쓰지 않는 기준)이다. 금액은 한국 원, 미국 달러(`$182.40`)다. 차트·지지선·저항선으로 공식을 바꾸지 않는다.

| 줄 | 값·표기 |
|---|---|
| 목표주가 | 채택한 목표가, 증권사·리포트 날짜·상향/하향/유지·상승여력 `upside_pct`·출처 |
| 가치기준 매수상한 | `buy_ceiling`과 채택한 EPS/BPS·비교배수(동종/자사 과거 중앙값)·대상 기간·출처 링크와 조회일 또는 기사 발행일·안전마진 20%. 여러 기준가치의 최솟값이면 그 사실을 적고 전체 산식은 실행 기록에 남긴다 |
| 수익실현 | `take_profit`, 괄호는 `(+N%)`로 `take_profit_pct` |
| 진입 | `entry` 뒤에 `이하`와 `entry_status`. 종가를 그대로 복사하지 않음 |
| 손절 | `stop`, 괄호는 `(−8%)` |

- **목표가 출처와 채택**: 30일 안의 증권사·애널리스트 목표가를 쓴다. 미국은 먼저 **신호 조회**의 `targets_30d`(Yahoo 투자의견 자료의 증권사·시각·기존/새 목표가)를 쓰고, 출처는 `[Yahoo Finance](https://finance.yahoo.com/quote/티커/analysis)`와 조회일로 표시한다. 그 밖에는 기사에서 찾고, 선정 기사에 없으면 `회사명 목표주가` / `티커 price target`으로 추가 검색한다. 여러 값을 확인하면 리포트 발표일(KST 날짜)이 가장 최근인 것, 같은 날이면 더 낮은 목표가, 그래도 같으면 증권사명 오름차순으로 정한다. 모든 확인값과 채택 이유를 저장하며 상승여력이 큰 값을 골라 쓰지 않는다. 확인하지 못하면 `• 목표주가 미확인`으로 쓰고 컨센서스 평균·오래된 값으로 채우지 않는다.
- 목표가는 보통 6~12개월 기준이라 4주 수익실현가로 쓰지 않는다. +16%·−8%는 진입가를 분모로 한 공통 계산 규칙이며 적정가치·기대수익률이 아니다. 실제 체결가가 다르면 비율을 체결가 기준으로 재산정해야 하며, 루틴은 주문하거나 체결을 가정해 기록하지 않는다.
- 매수상한·진입·수익실현은 내림, 손절은 올림이다(한국 KRX 호가 단위, 미국 1센트). 코드의 수익실현 `min(보수 기준가치, 진입가 × 1.16)`은 복원 호환용이며 현재 규칙에서는 진입가 × 1.16과 같다. **반드시 아래 코드로 계산**한다.

#### 종가 조회 실패 시

시세 조회의 최상위 `error`가 있으면 선정 기사에서 종가를 찾고, 없으면 `회사명 주가` / `티커 stock price`로 한 번 더 검색해 72시간 안의 기사에서 찾는다. 기사 종가의 거래일은 `expected_date`와 같아야 한다. `expected_date`가 없으면(달력 조회 실패) 시장별 휴장·거래일을 검색으로 확인하고 근거를 기록하며, 후보 간 날짜 비교나 주말만 빼는 계산으로 대신하지 않는다.

- 직전 거래일 종가를 확인하면 그 값으로 계산하고 가격 제목의 출처를 매체명으로 쓴다. 이때도 `trend` 결과(신뢰도 정상은 `true`, 신뢰도 낮음은 `error` 없는 `false`)와 저평가 기준일 정합성·진입가 입력을 확인해야 한다.
- 직전 거래일·종가·진입가 근거를 확인하지 못하면 신규 선정하지 않고 산출 불가 사유를 기록한다. 장중·오래된 종가·추정값으로 매매가를 계산하지 않는다.

### 가격 계산

각 `valuation_bases` 항목에는 `method`·`per_share`·`benchmark`와 아래 코드의 필수 `evidence` 항목을 넣는다. EPS/BPS와 비교배수의 출처·발행일은 각각 저장한다. 코드는 누락·날짜·통화·가격을 검사하는 계산기이며, 근거의 진위·정상화·기간 경계 시각·비교 가능성은 앞 단계에서 원문 검색 결과와 대조해야 한다.

```python
from datetime import date, timedelta
from decimal import Decimal, ROUND_CEILING, ROUND_FLOOR, ROUND_HALF_UP
from urllib.parse import urlsplit

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

def validate_entry_evidence(valuation_bases, market, brief_date):
    if not isinstance(valuation_bases, list) or not valuation_bases:
        raise ValueError("PER/PBR 기반 진입가 근거 필요")
    required = {"per_share_url", "per_share_date", "benchmark_url", "benchmark_date",
                "analyst", "period", "actual_or_forecast", "accounting_basis",
                "currency", "share_unit", "split_basis", "comparison_subject",
                "comparison_note", "valuation_date"}
    for basis in valuation_bases:
        evidence = basis.get("evidence") if isinstance(basis, dict) else None
        if not isinstance(evidence, dict) or any(
            not isinstance(evidence.get(key), str) or not evidence[key].strip()
            or evidence[key].strip().lower() in {"미확인", "unknown", "n/a"}
            for key in required
        ):
            raise ValueError("진입가의 출처·기간·비교 기준 메타데이터 필요")
        for key in ("per_share_url", "benchmark_url"):
            url = urlsplit(evidence[key])
            if url.scheme not in {"https", "http"} or not url.hostname or url.hostname == "news.google.com":
                raise ValueError("검색 결과에서 확인한 원문 URL 필요")
        for key in ("per_share_date", "benchmark_date"):
            published = date.fromisoformat(evidence[key])
            if not brief_date - timedelta(days=30) <= published <= brief_date:
                raise ValueError("진입가 근거 발행일의 30일 범위 확인 필요")
        if date.fromisoformat(evidence["valuation_date"]) >= brief_date:
            raise ValueError("평가 기준일은 직전 거래일까지여야 함")
        if evidence["currency"] != {"KR": "KRW", "US": "USD"}[market]:
            raise ValueError("종가와 주당 실적·자산의 통화 불일치")
        if evidence["actual_or_forecast"] not in {"actual", "forecast"}:
            raise ValueError("실제/예상 구분은 actual 또는 forecast 필요")

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

# per_share is EPS for PER, BPS for PBR; evidence preserves verified source metadata.
def levels(price, target, market, brief_date, valuation_bases):
    if market not in {"KR", "US"}:
        raise ValueError("시장은 KR 또는 US여야 함")
    validate_entry_evidence(valuation_bases, market, brief_date)
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

# 실제 실행은 검증한 valuation_bases로 levels(close, target_or_none, market,
# brief_date, valuation_bases)를 호출한다. 예시 숫자를 실행 입력으로 재사용하지 않는다.
```

## 시세 조회

후보를 확정하기 전에 `KR:종목코드` / `US:티커`를 한 번에 넘긴다. 종목마다 JSON 한 줄을 출력한다.

- 종가·등락률은 한국 네이버 우선, 실패 시 Yahoo Finance(`.KS` → `.KQ`), 미국은 Yahoo Finance다. 각 출처의 마지막 거래일이 `expected_date`와 같아야 성공이며, 오래된 응답은 오류로 남기고 다음 출처를 조회한다. 대체 출처로 성공해도 `warnings`의 이전 오류를 최종 기록에 포함한다.
- `expected_date`는 브리핑 날짜 전날까지의 최근 거래일이다(`exchange_calendars`의 `XKRX`·`XNYS`). 한국은 달력에 빠진 임시 휴장일을 거르려고 KOSPI 지수의 실제 거래일(네이버 → Yahoo `^KS11`)과 교차 확인한다. 달력 조회가 실패하면 종가를 성공으로 처리하지 않는다.
- `daily_change_pct`는 브리핑 날짜 − 7일 이후의 일간 등락률 표시값이고, `surge_excluded`는 반올림하지 않은 가격으로 비교한 결과다. `close_raw`는 원래 종가, 미국 `close`는 1센트로 반올림한 표시·계산용 값이다.
- `market_cap`은 한국 네이버 시세 API, 미국 Yahoo `marketCap` 제공값으로 종가와 독립적으로 조회한다. `market_cap.error`만 있으면 종가 실패로 보지 않고 `시가총액 미확인`을 표시한다.
- `trend`는 종가와 독립적으로 Yahoo `Adj Close`(한국 `.KS` → `.KQ`)의 과거 400일에서 마지막 80세션으로 계산한다. 서로 다른 출처·수정/비수정 계열을 이어 붙이지 않으며, 네이버 종가 성공도 추세 조회를 생략할 이유가 아니다. 오류는 `trend.error`, 이전 대체 출처 오류는 `trend.warnings`에 남는다.
- 스크립트는 종가·추세·시가총액 중 하나라도 오류인 종목을 전체 조회 후 60초 기다렸다가 한 번 더 조회한다. 두 번째 결과를 쓰고 첫 시도 오류는 `retry.first_errors`에 남긴다. `--discover-us`처럼 Yahoo를 대량 조회한 직후 실패가 몰릴 수 있기 때문이다.
- 재조회 뒤에도 한 시장 후보의 절반 이상(10개 이상)이 오류이면 **대량 조회 오류**로 본다. 개별 종목 문제로 판정하지 않고, 기사 검색 등 다른 작업을 먼저 한 뒤 실패 종목만 스크립트를 한 번 더 실행한다. 그래도 실패한 종목만 조회 오류로 보류하며 각 시도의 시각과 오류 원문을 기록한다.

Python 3.10 이상에서 설치·실행에 같은 인터프리터를 쓴다(아래 두 스크립트도 동일). 설치 실패는 기록하고 조회 스크립트는 계속 실행해 이용 가능한 결과를 남긴다.

```bash
python3 -m pip install -q yfinance beautifulsoup4 'exchange_calendars>=4.11.3'
# 첫 인수는 실행 시작 시 고정한 브리핑 날짜로, 나머지는 실제 후보로 바꾼다.
python3 - 2026-11-02 KR:005930 US:NVDA <<'EOF'
import ast, json, sys, time, urllib.request
from datetime import date, datetime, timedelta, timezone
from decimal import Decimal, ROUND_HALF_UP
from functools import lru_cache

KST = timezone(timedelta(hours=9))
today = date.fromisoformat(sys.argv[1])  # 실행 시작 시 고정한 브리핑 날짜
window_start = today - timedelta(days=7)  # 신호 기간(7일)이 걸친 첫 날짜

@lru_cache(maxsize=2)
def trading_calendar(market):
    import exchange_calendars as xcals
    if tuple(int(p) for p in xcals.__version__.split(".")[:3]) < (4, 11, 3):
        raise ValueError("exchange_calendars 4.11.3 이상 필요")
    name = {"KR": "XKRX", "US": "XNYS"}[market]
    # 일요일·연휴에도 조회 날짜가 달력의 마지막 세션을 넘지 않게 여유를 둔다.
    return xcals.get_calendar(name, start=today - timedelta(days=400), end=today + timedelta(days=14))

def kospi_dates():
    # exchange_calendars에 빠진 한국 임시 휴장일(예: 2026-06-03 지방선거, 2026-07-17 제헌절)을
    # 걸러내려고 KOSPI 지수가 실제로 거래된 날짜와 교차 확인한다.
    errors = []
    try:
        url = (f"https://api.finance.naver.com/siseJson.naver?symbol=KOSPI&requestType=1&timeframe=day"
               f"&startTime={today - timedelta(days=400):%Y%m%d}&endTime={today:%Y%m%d}")
        req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
        text = urllib.request.urlopen(req, timeout=10).read().decode("utf-8", "replace")
        dates = {datetime.strptime(r[0], "%Y%m%d").date() for r in ast.literal_eval(text.strip())[1:] if r[4]}
        if dates:
            return dates
        raise ValueError("데이터 없음")
    except Exception as e:
        errors.append(f"네이버 KOSPI: {type(e).__name__}: {e}"[:200])
    try:
        import yfinance as yf
        h = yf.Ticker("^KS11").history(start=(today - timedelta(days=400)).isoformat(),
                                       end=today.isoformat(), timeout=10)
        dates = {ts.date() for ts in h["Close"].dropna().index}
        if dates:
            return dates
        raise ValueError("데이터 없음")
    except Exception as e:
        errors.append(f"Yahoo ^KS11: {type(e).__name__}: {e}"[:200])
    raise ValueError("한국 실제 거래일 확인 실패: " + " / ".join(errors))

@lru_cache(maxsize=2)
def session_dates(market):
    # 브리핑 날짜 전날까지의 거래소 세션. 달력 첫 세션부터 읽어 휴장일 시작에 따른 범위 오류를 피한다.
    calendar = trading_calendar(market)
    sessions = [s.date() for s in calendar.sessions_in_range(
        calendar.first_session, calendar.date_to_session(today - timedelta(days=1), direction="previous"))]
    if market == "KR":
        traded = kospi_dates()
        last = max(d for d in traded if d < today)
        if last < sessions[-1] - timedelta(days=7):
            raise ValueError(f"KOSPI 거래일 자료가 오래됨: {last}")
        # 지수 자료가 없는 날은 휴장으로 본다. 07:00 기준이면 전날 지수는 이미 반영돼 있다.
        sessions = [d for d in sessions if d in traded]
    return sessions

def expected_session(market):
    return session_dates(market)[-1]

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
    return [(ts.date(), Decimal(str(c))) for ts, c in h["Close"].dropna().items()]

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
        dates = session_dates(market)[-80:]
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
            all_sessions = session_dates(market)
            changes, surge = {}, False
            threshold = Decimal("1.15") if market == "KR" else Decimal("1.20")
            for i, d in enumerate(all_sessions):
                if d < window_start or i == 0:
                    continue
                previous = all_sessions[i - 1]
                if d not in prices or previous not in prices:
                    raise ValueError(f"일간 등락률 거래일 데이터 누락: {previous} / {d}")
                p, c = prices[previous], prices[d]
                changes[d.isoformat()] = str(((c / p - 1) * 100).quantize(
                    Decimal("0.01"), rounding=ROUND_HALF_UP))
                surge = surge or c >= p * threshold
            return {**base, "source": name, "date": expected.isoformat(),
                    "close_raw": str(rows[-1][1]),
                    "close": str(rows[-1][1].quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)
                                 if market == "US" else rows[-1][1]),
                    "daily_change_pct": changes,
                    "surge_excluded": surge, "warnings": errors}
        except Exception as e:
            errors.append(f"{name}: {type(e).__name__}: {e}"[:200])
    return {**base, "error": errors}

RETRY_WAIT = 60  # 초. Yahoo 대량 조회 직후의 일시 실패를 피하려고 실패 종목만 한 번 다시 조회한다.

def lookup(market, code):
    result = quote(market, code)
    result["market_cap"] = market_cap(market, code)
    result["trend"] = trend(market, code)
    return result

def errors_of(result):
    return {k: v for k, v in {"quote": result.get("error"), "market_cap": result["market_cap"].get("error"),
                              "trend": result["trend"].get("error")}.items() if v}

targets = [arg.split(":", 1) for arg in sys.argv[2:]]
results = [lookup(market, code) for market, code in targets]
failed = [i for i, r in enumerate(results) if errors_of(r)]
if failed:
    time.sleep(RETRY_WAIT)
    for i in failed:
        first = errors_of(results[i])
        results[i] = {**lookup(*targets[i]), "retry": {"waited_sec": RETRY_WAIT, "first_errors": first}}
for result in results:
    print(json.dumps(result, ensure_ascii=False))
EOF
```

## 신호 조회

긍정 신호의 날짜·종류와 미국 목표가를 정형 데이터로 확인한다. 한국 DART 조회는 `DART_API_KEY`가 필요하며 키 값은 기록·출력·메시지에 남기지 않는다. 환경변수가 없고 작업 폴더에 `.env`(Git 제외, 형식은 `.env.example`)가 있으면 `set -a; . ./.env; set +a`로 불러온다. 키가 없거나 DART가 실패하면 `errors`에 남고 기사 검색으로 보충한다.

- `--discover-kr`: 신호 기간의 유가증권(`Y`)·코스닥(`K`) 주요사항·거래소 공시 중 자사주·배당·공급계약 분류를 종목별로 모으고, 네이버 종목분석 리포트 전체 목록에서 신호 기간 리포트(`research_reports`)가 있는 종목을 더한다. 리포트 종목의 시장은 네이버 종목 기본정보로 확인해 유가증권·코스닥만 후보로 넣고, 나머지(코넥스·확인 실패 포함)는 `research.skipped`에 남긴다. 페이지 한도로 일부만 조회하면 `errors`에 남는다.
- `--discover-us`: Wikipedia 구성표에서 S&P 500·S&P MidCap 400 종목(약 900개)을 받아 종목별 투자의견·목표가 상향을 8개 병렬로 조회하고, EPS 컨센서스 상회는 Yahoo 실적 캘린더(100건 단위 페이지)에서 찾는다. 종목별 실적 발표 조회를 수백 번 반복하면 Yahoo가 요청을 제한하기 때문이다. 실적 캘린더가 실패하면 60초 뒤 한 번 다시 조회하며, `earnings_calendar_events`가 `null`이 아니면 성공이고 `errors`의 `1차` 오류는 재시도 기록이다. 결과는 `candidates`(신호가 있는 종목)·`failed`·`errors`다.
- 후보 조회(`KR:종목코드` / `US:티커`):
  - 한국: `disclosures`(신호 기간 긍정 공시와 `title_only_eligible`·판정 사유), `disclosure_changes`(30일 자사주·배당·공급계약의 정정·취소 등 변경 가능 공시), `earnings_releases`(잠정실적 공시), `risk_disclosures`(30일 희석·소송·규제 공시), `research_reports`(신호 기간 네이버 리포트 목록). 각 항목에 날짜·제목·링크가 있다. `title_only_eligible`은 탐색에서는 `null`이며 후보별 완전 조회에서만 확정한다.
  - 미국: `target_raises`(신호 기간 상향)·`target_raise_score`, `target_cuts_30d`(30일 하향), `targets_30d`(30일 안의 목표가가 있는 모든 투자의견), `latest_earnings`(최근 실적 발표의 EPS 추정·실제·`beat`·`in_signal_window`).
- 공시 분류는 제목 문구 기준이다. 해지·철회·취소·해제·처분 공시와 `(자회사의 주요경영사항)` 공시는 긍정 신호로 분류하지 않는다. 자사주·배당·공급계약의 변경 가능 공시는 별도로 보존해 신규 결정의 자동 인정 여부를 점검한다. 같은 종류라는 사실만으로 원공시와 변경 공시를 같은 사건으로 연결하지 않으며, 애매한 제목은 기사로 확인한다.

```bash
# 첫 인수는 실행 시작 시 고정한 브리핑 날짜로, 나머지는 --discover-kr·--discover-us 또는 실제 후보로 바꾼다.
python3 - 2026-11-02 KR:005930 US:NVDA <<'EOF'
import io, json, os, sys, urllib.parse, urllib.request, zipfile
import xml.etree.ElementTree as ET
from datetime import date, datetime, time, timedelta, timezone
from decimal import Decimal
from functools import lru_cache

KST = timezone(timedelta(hours=9))
today = date.fromisoformat(sys.argv[1])  # 실행 시작 시 고정한 브리핑 날짜
cutoff = datetime.combine(today, time(7), KST)  # 브리핑 날짜 07:00 KST
signal_start = cutoff - timedelta(days=7)
risk_start = cutoff - timedelta(days=30)
DART_KEY = os.environ.get("DART_API_KEY", "").strip()

# 공시 분류와 제목만으로 인정할 신규 결정 목록을 구분한다.
CATEGORIES = [
    ("자사주", ("자기주식취득결정", "자기주식소각결정", "주식소각결정", "자기주식취득신탁계약체결결정")),
    ("배당", ("현금현물배당결정", "현금배당결정")),
    ("공급계약", ("단일판매공급계약체결",)),
    ("실적", ("영업잠정실적", "매출액또는손익구조")),
    ("희석", ("유상증자결정", "전환사채권발행결정", "신주인수권부사채권발행결정", "교환사채권발행결정")),
    ("소송", ("소송등의제기", "소송등의판결")),
    ("규제", ("불성실공시법인지정", "관리종목지정", "매매거래정지", "상장적격성실질심사")),
]
POSITIVE = {"자사주", "배당", "공급계약"}  # 잠정실적은 사건일 확인용이며 컨센서스 상회 신호가 아니다.
RISK = {"희석", "소송", "규제"}
CHANGES = ("정정", "변경", "연장", "추가", "재공시")
CANCELLATIONS = ("해지", "철회", "취소", "해제", "처분")
TITLE_ONLY = {"자사주": set(CATEGORIES[0][1]), "공급계약": set(CATEGORIES[2][1])}

def normalize(title):
    return "".join(ch for ch in title if ch not in " ㆍ·・()[]")

def category(title):
    t = normalize(title)
    if any(k in t for k in (*CANCELLATIONS, "자회사의주요경영사항")):
        return None
    return next((name for name, keys in CATEGORIES if any(k in t for k in keys)), None)

def positive_category(title):
    # 취소 공시도 원래 신호 종류를 남긴다. "체결" 대신 "해지"인 계약 제목도 포함한다.
    t = normalize(title)
    if "단일판매공급계약" in t:
        return "공급계약"
    if any(k in t for k in ("자기주식취득", "자기주식소각", "주식소각")):
        return "자사주"
    if any(k in t for k in CATEGORIES[1][1]):
        return "배당"
    return None

def new_decision_title(title):
    # 명시한 형식만 제거한다. 임의 문구를 지워 신규 결정으로 만들지 않는다.
    t = normalize(title)
    if t.startswith("주요사항보고서"):
        t = t[len("주요사항보고서"):]
    if t.endswith("자율공시"):
        t = t[:-len("자율공시")]
    return t

def inside(day, start):
    # 날짜만 있는 자료는 경계 날짜(시작일·브리핑 날짜)를 판단할 수 없으므로 인정하지 않는다.
    return start.date() < day < cutoff.date()

def dart(path, **params):
    if not DART_KEY:
        raise ValueError("DART_API_KEY 환경변수 없음")
    url = f"https://opendart.fss.or.kr/api/{path}?" + urllib.parse.urlencode({"crtfc_key": DART_KEY, **params})
    with urllib.request.urlopen(url, timeout=20) as response:
        return response.read()

@lru_cache(maxsize=1)
def corp_codes():
    with zipfile.ZipFile(io.BytesIO(dart("corpCode.xml"))) as z:
        root = ET.fromstring(z.read(z.namelist()[0]))
    return {(e.findtext("stock_code") or "").strip(): e.findtext("corp_code")
            for e in root.iter("list") if (e.findtext("stock_code") or "").strip()}

def dart_list(start, max_pages=10, **params):
    items, page, truncated = [], 1, False
    while True:
        data = json.loads(dart("list.json", bgn_de=f"{start:%Y%m%d}", end_de=f"{cutoff:%Y%m%d}",
                               page_no=page, page_count=100, **params))
        if data.get("status") == "013":  # 조회된 데이터 없음
            break
        if data.get("status") != "000":
            raise ValueError(f"DART {data.get('status')}: {data.get('message')}")
        items += data.get("list", [])
        if page >= int(data.get("total_page", 1)):
            break
        if page >= max_pages:
            truncated = True
            break
        page += 1
    return items, truncated

def disclosure(item):
    title = item["report_nm"].strip()
    t = normalize(title)
    return {"date": datetime.strptime(item["rcept_dt"], "%Y%m%d").date().isoformat(),
            "title": title, "category": category(title), "positive_category": positive_category(title),
            "is_amendment": any(k in t for k in CHANGES),
            "is_cancellation": any(k in t for k in CANCELLATIONS),
            "is_subsidiary": "자회사의주요경영사항" in t,
            "title_only_eligible": None, "title_only_reason": "후보별 완전 조회 필요",
            "filer": item.get("flr_nm"),
            "url": f"https://dart.fss.or.kr/dsaf001/main.do?rcpNo={item['rcept_no']}"}

def assess_title_signals(rows, complete):
    changes = [r for r in rows if r["positive_category"] in POSITIVE
               and not r["is_subsidiary"] and (r["is_amendment"] or r["is_cancellation"])]
    for r in rows:
        kind = r["positive_category"]
        # 제목이 같아도 서로 다른 계약일 수 있어 자동으로 원공시와 연결하지 않는다.
        later = [c for c in changes if c["positive_category"] == kind and c["date"] >= r["date"]]
        if not complete:
            reason = "DART 공시 목록 일부만 조회 — 후속 변경 확인 불가"
        elif not inside(date.fromisoformat(r["date"]), signal_start):
            reason = "신호 기간 밖 또는 경계 날짜"
        elif r["is_subsidiary"] or r["is_amendment"] or r["is_cancellation"]:
            reason = "자회사·정정·변경·취소 공시 — 신규 신호 자동 인정 불가"
        elif new_decision_title(r["title"]) not in TITLE_ONLY.get(kind, set()):
            reason = "신규 결정 제목 목록 밖 — 기사 확인 필요"
        elif later:
            reason = "같은 종류의 후속 변경 가능 공시 있음 — 원공시 관계 기사 확인 필요"
        else:
            reason = "신규 결정/체결 공시 제목 확인"
        r["title_only_eligible"] = reason == "신규 결정/체결 공시 제목 확인"
        r["title_only_reason"] = reason
        r["possible_followup_urls"] = [c["url"] for c in later]
    return changes

def naver_researches(code):
    req = urllib.request.Request(f"https://m.stock.naver.com/api/stock/{code}/integration",
                                 headers={"User-Agent": "Mozilla/5.0"})
    with urllib.request.urlopen(req, timeout=10) as response:
        data = json.load(response)
    rows = []
    for r in data.get("researches") or []:
        d = datetime.strptime(r["wdt"], "%Y%m%d").date()
        if inside(d, signal_start):
            rows.append({"date": d.isoformat(), "broker": r.get("bnm"), "title": r.get("tit"),
                         "url": f"https://finance.naver.com/research/company_read.naver?nid={r['id']}"})
    return rows

def kr(code):
    out = {"source": "DART OpenAPI · 네이버 금융", "errors": []}
    try:
        corp = corp_codes().get(code)
        if not corp:
            raise ValueError("DART 고유번호 없음")
        items, truncated = dart_list(risk_start, corp_code=corp)
        rows = [disclosure(i) for i in items]
        out["disclosure_changes"] = assess_title_signals(rows, complete=not truncated)
        out["disclosures"] = [r for r in rows if r["category"] in POSITIVE
                              and inside(date.fromisoformat(r["date"]), signal_start)]
        out["earnings_releases"] = [r for r in rows if r["category"] == "실적"
                                    and inside(date.fromisoformat(r["date"]), signal_start)]
        out["risk_disclosures"] = [r for r in rows if r["category"] in RISK]
        if truncated:
            out["errors"].append("DART 공시 목록 일부만 조회")
    except Exception as e:
        out["disclosures"] = out["disclosure_changes"] = out["earnings_releases"] = out["risk_disclosures"] = None
        out["errors"].append(f"DART: {type(e).__name__}: {e}"[:200])
    try:
        out["research_reports"] = naver_researches(code)
    except Exception as e:
        out["research_reports"] = None
        out["errors"].append(f"네이버 리포트: {type(e).__name__}: {e}"[:200])
    return out

def us(code, earnings=True):
    import yfinance as yf
    t = yf.Ticker(code.replace(".", "-"))
    out = {"source": "Yahoo Finance", "errors": []}
    try:
        changes, cuts, targets = [], [], []
        for ts, r in t.upgrades_downgrades.iterrows():
            when = ts.to_pydatetime()
            when = (when.replace(tzinfo=timezone.utc) if when.tzinfo is None else when).astimezone(KST)
            if not risk_start < when <= cutoff:
                continue
            prior, current = Decimal(str(r["priorPriceTarget"] or 0)), Decimal(str(r["currentPriceTarget"] or 0))
            row = {"at": when.isoformat(timespec="minutes"), "firm": r["Firm"], "action": r["Action"],
                   "from_grade": r["FromGrade"], "to_grade": r["ToGrade"],
                   "prior_target": str(prior) if prior > 0 else None,
                   "current_target": str(current) if current > 0 else None}
            if current > 0:  # 가격 블록의 목표주가 후보(상향·하향·유지·최초 커버리지 모두)
                targets.append(row)
            raised = r["Action"] == "up" or (r["Action"] != "init" and prior > 0 and current > prior)
            lowered = r["Action"] == "down" or (prior > 0 and 0 < current < prior)
            if raised and when > signal_start:
                changes.append(row)
            if lowered:
                cuts.append(row)
        out["target_raises"] = changes
        out["target_cuts_30d"] = cuts
        out["targets_30d"] = targets
        firms = {c["firm"] for c in changes}
        out["target_raise_score"] = 2 if len(firms) >= 2 else len(firms)
    except Exception as e:
        out["target_raises"] = out["target_cuts_30d"] = out["targets_30d"] = None
        out["errors"].append(f"투자의견: {type(e).__name__}: {e}"[:200])
    if not earnings:  # 시장 탐색은 실적 캘린더로 따로 확인한다.
        return out
    try:
        latest = None
        for ts, r in t.get_earnings_dates(limit=8).iterrows():
            when = ts.to_pydatetime().astimezone(KST)
            est, rep = r.get("EPS Estimate"), r.get("Reported EPS")
            if when <= cutoff and est == est and rep == rep and est is not None and rep is not None:
                latest = {"at": when.isoformat(timespec="minutes"), "eps_estimate": str(est),
                          "eps_reported": str(rep), "surprise_pct": str(r.get("Surprise(%)")),
                          "basis": "Yahoo 제공 EPS(추정·실제 동일 제공사 기준)",
                          "in_signal_window": when > signal_start,
                          "beat": Decimal(str(rep)) > Decimal(str(est))}
                break
        out["latest_earnings"] = latest
    except Exception as e:
        out["latest_earnings"] = None
        out["errors"].append(f"실적 발표: {type(e).__name__}: {e}"[:200])
    return out

def discover_kr():
    # 시장 탐색: 신호 기간의 유가증권·코스닥 공시에서 긍정 신호 분류만 모은다.
    found, errors = {}, []
    for corp_cls in ("Y", "K"):
        for kind in ("B", "I"):  # 주요사항보고, 거래소 공시
            try:
                items, truncated = dart_list(signal_start, max_pages=20, corp_cls=corp_cls, pblntf_ty=kind)
                if truncated:
                    errors.append(f"{corp_cls}/{kind}: 20페이지까지만 조회")
                for i in items:
                    row = disclosure(i)
                    if row["category"] in POSITIVE and inside(date.fromisoformat(row["date"]), signal_start):
                        found.setdefault(i["stock_code"], {"code": i["stock_code"], "name": i["corp_name"],
                                                           "market": corp_cls, "disclosures": []})["disclosures"].append(row)
            except Exception as e:
                errors.append(f"{corp_cls}/{kind}: {type(e).__name__}: {e}"[:200])
    try:
        reports, truncated = research_list()
        if truncated:
            errors.append("네이버 리포트: 20페이지까지만 조회")
        by_code = {}
        for r in reports:
            by_code.setdefault(r["code"], []).append(r)
        new = [c for c in by_code if c not in found]
        from concurrent.futures import ThreadPoolExecutor
        with ThreadPoolExecutor(max_workers=8) as pool:
            markets = dict(zip(new, pool.map(listing_market, new)))
        skipped = []
        for code, rows in by_code.items():
            if code in found:
                found[code]["research_reports"] = rows
                continue
            market = markets[code]
            if market in ("Y", "K"):
                found[code] = {"code": code, "name": rows[0]["name"], "market": market,
                               "disclosures": [], "research_reports": rows}
            else:
                skipped.append({"code": code, "name": rows[0]["name"], "market": market})
        research = {"reports": len(reports), "companies": len(by_code), "skipped": skipped}
    except Exception as e:
        research = None
        errors.append(f"네이버 리포트: {type(e).__name__}: {e}"[:200])
    return {"mode": "discover", "window": [signal_start.isoformat(), cutoff.isoformat()],
            "candidates": sorted(found.values(), key=lambda c: c["code"]), "research": research, "errors": errors}

def research_list(max_pages=20):
    # 네이버 종목분석 리포트 전체 목록(최신순)에서 신호 기간 리포트만 모은다. 상향 여부는 기사로 확인한다.
    rows, page = [], 1
    while True:
        req = urllib.request.Request(f"https://m.stock.naver.com/api/research/company?page={page}&pageSize=100",
                                     headers={"User-Agent": "Mozilla/5.0"})
        with urllib.request.urlopen(req, timeout=10) as response:
            data = json.load(response)
        for r in data:
            d = date.fromisoformat(r["writeDate"])
            if r.get("itemCode") and inside(d, signal_start):
                rows.append({"code": r["itemCode"], "name": r.get("itemName"), "date": d.isoformat(),
                             "broker": r.get("brokerName"), "title": r.get("title"), "url": r.get("endUrl")})
        if len(data) < 100 or date.fromisoformat(data[-1]["writeDate"]) <= signal_start.date():
            return rows, False
        if page >= max_pages:
            return rows, True
        page += 1

def listing_market(code):
    # 리포트 목록에는 시장 구분이 없어 네이버 종목 기본정보로 유가증권(Y)·코스닥(K)을 확인한다.
    try:
        req = urllib.request.Request(f"https://m.stock.naver.com/api/stock/{code}/basic",
                                     headers={"User-Agent": "Mozilla/5.0"})
        with urllib.request.urlopen(req, timeout=10) as response:
            name = (json.load(response).get("stockExchangeType") or {}).get("name")
        return {"KOSPI": "Y", "KOSDAQ": "K"}.get(name, name)
    except Exception as e:
        return f"확인 실패: {type(e).__name__}: {e}"[:200]

US_INDEXES = [("S&P 500", "https://en.wikipedia.org/wiki/List_of_S%26P_500_companies"),
              ("S&P MidCap 400", "https://en.wikipedia.org/wiki/List_of_S%26P_400_companies")]

def us_universe():
    # 탐색 범위: S&P 500·S&P MidCap 400 구성 종목(Wikipedia 구성표, 실행 시점 기준).
    from bs4 import BeautifulSoup
    tickers, errors = {}, []
    for name, url in US_INDEXES:
        try:
            req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0 (daily-stock-brief)"})
            with urllib.request.urlopen(req, timeout=20) as response:
                table = BeautifulSoup(response.read(), "html.parser").find("table", id="constituents")
            rows = table.find_all("tr")
            headers = [c.get_text(strip=True) for c in rows[0].find_all(["th", "td"])]
            col = next(i for i, h in enumerate(headers) if h in ("Symbol", "Ticker"))
            found = 0
            for tr in rows[1:]:
                cells = tr.find_all(["td", "th"])
                symbol = cells[col].get_text(strip=True) if len(cells) > col else ""
                if symbol:
                    tickers.setdefault(symbol, set()).add(name)
                    found += 1
            if found < 300:
                raise ValueError(f"구성 종목 {found}개만 확인")
        except Exception as e:
            errors.append(f"{name} 구성 종목: {type(e).__name__}: {e}"[:200])
    return tickers, errors

def earnings_beats():
    # 신호 기간의 실적 발표 전체를 Yahoo 실적 캘린더에서 페이지 단위로 받아 EPS 컨센서스 상회만 남긴다.
    import yfinance as yf
    calendar = yf.Calendars(start=signal_start.date().isoformat(), end=(today + timedelta(days=1)).isoformat())
    beats, offset = {}, 0
    while True:
        page = calendar.get_earnings_calendar(filter_most_active=False, limit=100, offset=offset, force=True)
        for symbol, r in page.iterrows():
            when = r["Event Start Date"].to_pydatetime().astimezone(KST)
            est, rep = r.get("EPS Estimate"), r.get("Reported EPS")
            if signal_start < when <= cutoff and est == est and rep == rep and est is not None and rep is not None \
                    and Decimal(str(rep)) > Decimal(str(est)):
                beats[symbol] = {"at": when.isoformat(timespec="minutes"), "eps_estimate": str(est),
                                 "eps_reported": str(rep), "surprise_pct": str(r.get("Surprise(%)")),
                                 "basis": "Yahoo 제공 EPS(추정·실제 동일 제공사 기준)",
                                 "in_signal_window": True, "beat": True}
        offset += len(page)
        if len(page) < 100:
            return beats, offset
        if offset >= 5000:
            raise ValueError("실적 캘린더 5,000건 초과 — 일부만 조회")

def discover_us():
    # 시장 탐색: 구성 종목의 신호 기간 투자의견·목표가 상향(종목별 조회)과 실적 컨센서스 상회(캘린더)를 모은다.
    from concurrent.futures import ThreadPoolExecutor
    import yfinance  # noqa: F401 — 스레드에서 동시에 처음 import하면 덜 초기화된 모듈이 보일 수 있다.
    from time import sleep
    universe, errors = us_universe()
    beats, scanned_events, calendar_errors = None, None, []
    for attempt in range(2):  # 시세 조회와 같이 60초 뒤 한 번 다시 조회하고 첫 시도 오류도 남긴다.
        try:
            beats, scanned_events = earnings_beats()
            break
        except Exception as e:
            calendar_errors.append(f"실적 캘린더 {attempt + 1}차: {type(e).__name__}: {e}"[:200])
            if attempt == 0:
                sleep(60)
    errors += calendar_errors
    def scan(code):
        try:
            return code, us(code, earnings=False)
        except Exception as e:
            return code, {"errors": [f"{type(e).__name__}: {e}"[:200]]}
    found, failed = [], {}
    with ThreadPoolExecutor(max_workers=8) as pool:
        for code, r in pool.map(scan, sorted(universe)):
            if r.get("errors"):
                failed[code] = r["errors"]
            beat = (beats or {}).get(code.replace(".", "-"))
            if r.get("target_raise_score") or beat:
                found.append({"code": code, "indexes": sorted(universe[code]),
                              "target_raises": r.get("target_raises"),
                              "target_raise_score": r.get("target_raise_score"), "latest_earnings": beat})
    if failed:
        errors.append(f"투자의견 조회 실패 {len(failed)}/{len(universe)}개 — 해당 종목의 상향 신호는 미확인")
    return {"mode": "discover", "window": [signal_start.isoformat(), cutoff.isoformat()],
            "universe": {"indexes": [n for n, _ in US_INDEXES], "size": len(universe)},
            "earnings_calendar_events": scanned_events, "candidates": found, "failed": failed, "errors": errors}

if sys.argv[2:] == ["--discover-kr"]:
    print(json.dumps(discover_kr(), ensure_ascii=False))
elif sys.argv[2:] == ["--discover-us"]:
    print(json.dumps(discover_us(), ensure_ascii=False))
else:
    for arg in sys.argv[2:]:
        market, code = arg.split(":", 1)
        base = {"market": market, "code": code, "signal_window": [signal_start.isoformat(), cutoff.isoformat()]}
        try:
            result = kr(code) if market == "KR" else us(code)
        except Exception as e:
            result = {"error": f"{type(e).__name__}: {e}"[:200]}
        print(json.dumps({**base, **result}, ensure_ascii=False))
EOF
```

## 가치·실적 지표 조회

시세 조회를 통과한 후보 전체를 한 번에 넘긴다(8개씩 병렬, 동종·업종 조회는 실행 안에서 캐시, 입력 순서대로 출력). 인수는 `KR:종목코드=종가@종가기준일` / `US:티커=종가@종가기준일`이며 시세 조회의 `close`·`date`를 그대로 쓴다.

- `eps`·`bps`: 한국은 네이버의 최근 4분기 합산 EPS·최근 분기 BPS, 미국은 Yahoo `trailingEps`·`bookValue`다. 한국 `cnsEps`는 참고용 추정치다.
- `peers`·`history`: 동종 종목별 PER·PBR과 최근 3개 실제 회계연도 PER. 한국은 네이버 연간 PER·EPS·ROE, 미국은 Yahoo 연간 희석 EPS·결산일 이전 마지막 종가(`close_date`)와 그 비율이며 ROE는 최상위 `roe`다. 적자 연도는 표본에서 빠진다. 표본 연도의 연간 EPS가 최근 4분기 EPS의 절반 미만이거나 확인되지 않으면 자사 과거 비교는 `passed: null`이다(이익 회복기·급증으로 부풀려진 과거 배수 방지, 절반은 운영 기준).
- `valuation.comparisons`: 비교별 자사 배수·표본 수·중앙값·할인율·`passed`, 통과 비교의 `valuation_basis`(`levels()` 입력).
- `quarters`·`sustainability`: Yahoo 분기 영업이익·영업현금흐름·FCF(최근 5분기)와 최신 분기 판정·`yoy`·`ocf_negative_quarters_4q`·등급.
- 오류는 최상위 `error`(조회 실패), `errors`(동종·연간·분기 일부 실패), `sustainability.error`로 남는다. 일부 실패는 해당 비교만 미확인이다. 제공값은 조회 시점 값이므로 조회 시각을 함께 저장한다.

```bash
# 첫 인수는 실행 시작 시 고정한 브리핑 날짜로, 나머지는 실제 후보와 시세 조회 결과로 바꾼다.
python3 - 2026-11-02 KR:005930=71200@2026-10-30 US:NVDA=182.4@2026-10-30 <<'EOF'
import json, statistics, sys, urllib.request
from datetime import date, datetime, timedelta, timezone
from decimal import Decimal, InvalidOperation
from functools import lru_cache

KST = timezone(timedelta(hours=9))
today = date.fromisoformat(sys.argv[1])  # 실행 시작 시 고정한 브리핑 날짜
MAX_DISCOUNT = Decimal("0.5")

def num(value):
    # 제공값 문자열("9.26배", "22,292원", "N/A")을 Decimal로 바꾸고, 없거나 유한하지 않으면 None.
    if value is None or isinstance(value, bool):
        return None
    text = str(value).replace(",", "").replace("배", "").replace("원", "").replace("%", "").strip()
    try:
        d = Decimal(text)
    except InvalidOperation:
        return None
    return d if d.is_finite() else None

def positive(value):
    d = num(value)
    return d if d is not None and d > 0 else None

def median(values):
    return statistics.median(values) if values else None

def naver_json(path):
    req = urllib.request.Request(f"https://m.stock.naver.com/api/stock/{path}",
                                 headers={"User-Agent": "Mozilla/5.0"})
    with urllib.request.urlopen(req, timeout=10) as response:
        return json.load(response)

# 같은 업종 후보가 많으면 동종 종목 조회가 겹치므로 실행 안에서 캐시한다.
@lru_cache(maxsize=None)
def naver_infos(code):
    data = naver_json(f"{code}/integration")
    infos = {i.get("code"): i for i in data.get("totalInfos", [])}
    return data, infos

def yahoo_quarters(symbol):
    import yfinance as yf
    t = yf.Ticker(symbol)
    cf, inc = t.quarterly_cashflow, t.quarterly_income_stmt
    rows = []
    for col in sorted(set(cf.columns) & set(inc.columns), reverse=True)[:5]:
        def cell(frame, key):
            if key not in frame.index:
                return None
            v = frame.loc[key, col]
            return None if v != v else str(Decimal(str(v)).quantize(Decimal("1")))
        rows.append({"period_end": col.date().isoformat(),
                     "operating_income": cell(inc, "Operating Income"),
                     "operating_cash_flow": cell(cf, "Operating Cash Flow"),
                     "free_cash_flow": cell(cf, "Free Cash Flow")})
    return rows

def sustainability(quarters):
    # 최신 분기와 4분기 전(전년 동기)을 같은 출처·같은 정의로 비교한다.
    if not quarters:
        return {"eligible": None, "error": "분기 실적 없음"}
    latest = quarters[0]
    if date.fromisoformat(latest["period_end"]) < today - timedelta(days=200):
        return {"eligible": None, "error": f"최신 분기 자료가 오래됨: {latest['period_end']}"}
    oi, ocf = num(latest["operating_income"]), num(latest["operating_cash_flow"])
    if oi is None or ocf is None:
        return {"eligible": None, "error": "영업이익 또는 영업현금흐름 누락"}
    result = {"period_end": latest["period_end"], "operating_income_positive": oi > 0,
              "operating_cash_flow_positive": ocf > 0, "eligible": oi > 0 and ocf > 0, "yoy": None}
    prior = next((q for q in quarters[1:] if abs((date.fromisoformat(latest["period_end"])
            - date.fromisoformat(q["period_end"])).days - 365) <= 10), None)
    if prior and num(prior["operating_income"]) is not None and num(prior["operating_cash_flow"]) is not None:
        result["yoy"] = {"period_end": prior["period_end"],
                         "operating_income_maintained": oi >= num(prior["operating_income"]),
                         "operating_cash_flow_maintained": ocf >= num(prior["operating_cash_flow"])}
    # 현금흐름이 분기마다 크게 오가는 업종을 가려내려고 최근 4분기의 유출 횟수를 함께 본다(합산하지 않음).
    recent = [num(q["operating_cash_flow"]) for q in quarters[:4]]
    result["ocf_negative_quarters_4q"] = (sum(v < 0 for v in recent)
                                          if len(recent) == 4 and None not in recent else None)
    yoy = result["yoy"]
    result["grade"] = (0 if not result["eligible"] else
                       2 if yoy and yoy["operating_income_maintained"] and yoy["operating_cash_flow_maintained"]
                       and result["ocf_negative_quarters_4q"] == 0 else 1)
    return result

def kr(code):
    out = {"source": "네이버 금융", "errors": []}
    data, infos = naver_infos(code)
    out["industry_code"] = data.get("industryCode")
    for key in ("eps", "bps", "cnsEps"):
        i = infos.get(key, {})
        out[key] = {"value": str(num(i.get("value"))) if num(i.get("value")) is not None else None,
                    "period": (i.get("valueDesc") or "").rstrip(".") or None,
                    "basis": "추정" if key == "cnsEps" else "실제 최근 4분기 합산"}
    peers = []
    for p in data.get("industryCompareInfo") or []:
        pc = p.get("itemCode")
        if not pc or pc == code:
            continue
        try:
            _, pi = naver_infos(pc)
            peers.append({"code": pc, "name": p.get("stockName"),
                          "per": str(positive(pi.get("per", {}).get("value")) or "") or None,
                          "pbr": str(positive(pi.get("pbr", {}).get("value")) or "") or None,
                          "per_period": (pi.get("per", {}).get("valueDesc") or "").rstrip(".") or None})
        except Exception as e:
            out["errors"].append(f"동종 {pc}: {type(e).__name__}: {e}"[:200])
    out["peers"] = peers
    try:
        fi = naver_json(f"{code}/finance/annual")["financeInfo"]
        actual = [t["key"] for t in fi["trTitleList"] if t.get("isConsensus") == "N"][-3:]
        rows = {r["title"]: r["columns"] for r in fi["rowList"]}
        out["history"] = [{"fiscal_year_end": k, "per": rows.get("PER", {}).get(k, {}).get("value"),
                           "eps": rows.get("EPS", {}).get(k, {}).get("value"),
                           "roe": rows.get("ROE", {}).get(k, {}).get("value")} for k in actual]
    except Exception as e:
        out["history"] = []
        out["errors"].append(f"연간 재무: {type(e).__name__}: {e}"[:200])
    out["urls"] = {"display": f"https://finance.naver.com/item/main.naver?code={code}",
                   "per_share": f"https://m.stock.naver.com/api/stock/{code}/integration",
                   "peers": f"https://m.stock.naver.com/api/stock/{code}/integration",
                   "history": f"https://m.stock.naver.com/api/stock/{code}/finance/annual"}
    return out

@lru_cache(maxsize=None)
def yahoo_info(symbol):
    import yfinance as yf
    return yf.Ticker(symbol).get_info()

@lru_cache(maxsize=None)
def industry_top(industry_key):
    import yfinance as yf
    return list(yf.Industry(industry_key).top_companies.index)

def us(code):
    import yfinance as yf
    symbol = code.replace(".", "-")
    out = {"source": "Yahoo Finance", "errors": []}
    info = yahoo_info(symbol)
    if info.get("financialCurrency") != "USD":
        raise ValueError(f"재무 통화 불일치: {info.get('financialCurrency')!r}")
    out["industry_code"] = info.get("industryKey")
    out["eps"] = {"value": str(info["trailingEps"]) if info.get("trailingEps") is not None else None,
                  "period": "TTM", "basis": "실제 최근 4분기 합산(GAAP)"}
    out["bps"] = {"value": str(info["bookValue"]) if info.get("bookValue") is not None else None,
                  "period": "최근 분기", "basis": "실제"}
    out["roe"] = str(info.get("returnOnEquity")) if info.get("returnOnEquity") is not None else None
    peers = []
    try:
        for pc in [s for s in industry_top(info["industryKey"]) if s != symbol][:8]:
            try:
                pi = yahoo_info(pc)
                if pi.get("financialCurrency") != "USD":
                    continue
                peers.append({"code": pc, "name": pi.get("shortName"),
                              "per": str(positive(pi.get("trailingPE")) or "") or None,
                              "pbr": str(positive(pi.get("priceToBook")) or "") or None, "per_period": "TTM"})
            except Exception as e:
                out["errors"].append(f"동종 {pc}: {type(e).__name__}: {e}"[:200])
    except Exception as e:
        out["errors"].append(f"동종 목록: {type(e).__name__}: {e}"[:200])
    out["peers"] = peers
    out["history"] = []
    try:
        # Yahoo는 과거 PER을 직접 주지 않으므로 최근 3개 회계연도마다 결산일 이전 마지막 종가 ÷ 연간 희석 EPS로 만든다.
        # 두 값 모두 Yahoo가 분할을 반영한 계열이다(auto_adjust=False의 Close는 분할만 반영, 배당 미반영).
        t = yf.Ticker(symbol)
        inc = t.income_stmt
        if "Diluted EPS" not in inc.index:
            raise ValueError("연간 희석 EPS 없음")
        years = sorted(inc.columns, reverse=True)[:3]
        closes = t.history(start=(min(years) - timedelta(days=10)).date().isoformat(),
                           end=(max(years) + timedelta(days=1)).date().isoformat(),
                           auto_adjust=False, timeout=10)["Close"].dropna()
        for fy in years:
            eps = positive(inc.loc["Diluted EPS", fy])
            before = closes[closes.index.date <= fy.date()]
            close_at = positive(before.iloc[-1]) if len(before) else None
            out["history"].append({"fiscal_year_end": fy.date().isoformat(),
                                   "per": str(close_at / eps) if eps and close_at else None,
                                   "eps": str(eps) if eps else None,
                                   "close": str(close_at) if close_at else None,
                                   "close_date": before.index[-1].date().isoformat() if len(before) else None,
                                   "roe": None})
    except Exception as e:
        out["errors"].append(f"연간 PER: {type(e).__name__}: {e}"[:200])
    out["urls"] = {"display": f"https://finance.yahoo.com/quote/{symbol}/key-statistics",
                   "per_share": f"https://finance.yahoo.com/quote/{symbol}/key-statistics",
                   "peers": f"https://finance.yahoo.com/sectors/{info.get('sectorKey')}/{info.get('industryKey')}",
                   "history": f"https://finance.yahoo.com/quote/{symbol}/financials"}
    return out

def compare(market, code, close, close_date, f):
    close = positive(close)
    eps, bps = positive(f["eps"]["value"]), positive(f["bps"]["value"])
    peer_per = [d for d in (positive(p["per"]) for p in f["peers"]) if d is not None]
    peer_pbr = [d for d in (positive(p["pbr"]) for p in f["peers"]) if d is not None]
    hist_per = [d for d in (positive(h["per"]) for h in f["history"]) if d is not None]
    result = {"close": str(close), "per_close": None, "pbr_close": None, "comparisons": []}
    if close is None:
        result["error"] = "종가 없음"
        return result
    retrieved = datetime.now(KST).date().isoformat()
    names = ", ".join(p["name"] or p["code"] for p in f["peers"])[:200]
    def add(kind, metric, own, values, minimum, per_share, benchmark_url, subject, unverified=None):
        if own is None:
            return
        entry = {"kind": kind, "metric": metric, "own": str(own), "sample": len(values)}
        if len(values) < minimum:
            entry.update(passed=None, reason=f"유효 비교값 {len(values)}개 (필요 {minimum}개)")
        else:
            bench = median(values)
            discount = 1 - own / bench
            entry.update(benchmark=str(bench), discount=str(discount), passed=own < bench)
        if entry.get("passed") and discount >= MAX_DISCOUNT:
            # 제공사 동종 분류·일회성 이익 때문에 생기는 과도한 격차는 저평가 근거로 쓰지 않는다.
            entry.update(passed=None, reason=f"할인율 {MAX_DISCOUNT:.0%} 이상 — 비교 가능성 미확인")
        elif entry.get("passed") and unverified:
            entry.update(passed=None, reason=unverified)
        elif entry.get("passed"):
            # levels()에 그대로 넘길 수 있는 기준가치 입력. 채택 여부는 지침의 지표 선택 규칙을 따른다.
            entry["valuation_basis"] = {"method": metric, "per_share": str(per_share), "benchmark": str(bench),
                "evidence": {"per_share_url": f["urls"]["per_share"], "per_share_date": retrieved,
                             "benchmark_url": benchmark_url, "benchmark_date": retrieved,
                             "analyst": f"{f['source']} 제공값",
                             "period": f"{f['eps' if metric == 'PER' else 'bps']['period']} "
                                       f"({f['eps' if metric == 'PER' else 'bps']['basis']})",
                             "actual_or_forecast": "actual",
                             "accounting_basis": "제공사 표시 기준" if market == "KR" else "GAAP",
                             "currency": "KRW" if market == "KR" else "USD", "share_unit": "보통주 1주",
                             "split_basis": "제공사 현재 주식 수 기준", "comparison_subject": subject,
                             "comparison_note": f"{kind} {len(values)}개 · 자사 배수 = 직전 종가 ÷ 제공 주당값",
                             "valuation_date": close_date}}
        result["comparisons"].append(entry)
    per_close = close / eps if eps else None
    pbr_close = close / bps if bps else None
    result["per_close"] = str(per_close) if per_close else None
    result["pbr_close"] = str(pbr_close) if pbr_close else None
    years = ", ".join(h["fiscal_year_end"] for h in f["history"])
    # 이익이 크게 낮았던 해의 PER은 부풀려져 있어 회복·급증 국면(이익 정점 포함)의 기업을 싸게 보이게 하므로 과거 비교에서 쓰지 않는다.
    hist_note = None
    for h in f["history"]:
        if positive(h["per"]) is None:
            continue
        year_eps = num(h.get("eps"))
        if year_eps is None:
            hist_note = f"{h['fiscal_year_end']} 연간 EPS 미확인 — 이익 회복기·급증 여부 확인 불가"
            break
        if eps and year_eps < eps / 2:
            hist_note = f"{h['fiscal_year_end']} 연간 EPS {year_eps} < 최근 4분기 EPS {eps}의 절반 — 이익 회복기·급증 배수"
            break
    add("동종 중앙값", "PER", per_close, peer_per, 3, eps, f["urls"]["peers"], names)
    add("자사 최근 3개 회계연도 중앙값", "PER", per_close, hist_per, 3, eps, f["urls"]["history"], f"자사 {years}",
        unverified=hist_note)
    add("동종 중앙값", "PBR", pbr_close, peer_pbr, 3, bps, f["urls"]["peers"], names)
    return result

def run(market, code, close, close_date):
    base = {"market": market, "code": code,
            "retrieved_at": datetime.now(KST).isoformat(timespec="seconds")}
    try:
        f = kr(code) if market == "KR" else us(code)
    except Exception as e:
        return {**base, "error": f"{type(e).__name__}: {e}"[:200]}
    symbols = [f"{code}.KS", f"{code}.KQ"] if market == "KR" else [code.replace(".", "-")]
    quarters = []
    for s in symbols:
        try:
            quarters = yahoo_quarters(s)
            if quarters:
                break
        except Exception as e:
            f["errors"].append(f"분기 실적 {s}: {type(e).__name__}: {e}"[:200])
    return {**base, **f, "quarters": quarters, "quarters_source": "Yahoo Finance",
            "sustainability": sustainability(quarters), "valuation": compare(market, code, close, close_date, f)}

def run_arg(arg):
    # 형식: KR:005930=종가@종가기준일 (시세 조회의 close·date를 그대로 넘긴다)
    target, rest = arg.split("=", 1)
    close, close_date = rest.split("@", 1)
    market, code = target.split(":", 1)
    return run(market, code, close, close_date)

# 빠른 확인에서 후보 전체를 넘기므로 8개씩 병렬 조회하고 입력 순서대로 출력한다.
from concurrent.futures import ThreadPoolExecutor
import yfinance  # noqa: F401 — 스레드에서 동시에 처음 import하지 않도록 미리 불러온다.
with ThreadPoolExecutor(max_workers=8) as pool:
    for result in pool.map(run_arg, sys.argv[2:]):
        print(json.dumps(result, ensure_ascii=False), flush=True)
EOF
```

## 실행 정보

부모가 있고 이번 실행에 필요한 섹션의 게시 시도를 마쳤으면(섹션 답글이 3번 실패했어도) 실행 정보를 마지막 답글로 보낸다. 이미 있으면 중복 게시하지 않는다. 섹션은 모두 있고 실행 정보만 빠졌다면 일치하는 스냅샷의 원래 진단으로 게시하고 `실행 정보 누락 복구`를 밝힌다. 원래 값을 복원할 수 없으면 토큰·모델·소요 시간은 `미확인`, 조사 진단은 `원래 조사 기록 복원 불가`로 쓰며 새 실행의 값으로 대체하지 않는다.

신규 실행에서는 현재 도구 문맥에서 확인한 **현재 세션 ID·메인 JSONL 경로**와 **실제 시작 시각**을 아래 명령의 인수로 넘긴다. 확인할 수 없으면 토큰·응답·모델을 `집계 불가 (현재 세션 기록 미확인)`으로 쓰고, 파일 수정 시각으로 다른 세션을 고르지 않는다.

```bash
python3 - /확인한/현재/세션ID.jsonl 세션ID 2026-11-02T07:04:00+09:00 <<'EOF'
import json, sys
from datetime import datetime, timezone, timedelta
from pathlib import Path

KST = timezone(timedelta(hours=9))
def parse_time(value):
    result = datetime.fromisoformat(value.replace("Z", "+00:00"))
    if result.tzinfo is None:
        raise ValueError("시각에 시간대 필요")
    return result

def read(path, session_id, started_at):
    usage, models = {}, set()
    with open(path, encoding="utf-8") as f:
        for n, line in enumerate(f):
            try:
                e = json.loads(line)
            except ValueError:
                continue
            m = e.get("message")
            if e.get("type") == "assistant" and isinstance(m, dict) and m.get("usage"):
                if e.get("sessionId") != session_id:
                    raise ValueError("집계 대상의 세션 ID 불일치")
                if parse_time(e["timestamp"]) < started_at:
                    continue  # 재개된 세션의 이전 실행은 제외
                usage[m.get("id") or n] = m["usage"]
                if m.get("model"):
                    models.add(m["model"])
    return usage, models

try:
    main, session_id = Path(sys.argv[1]).expanduser().resolve(), sys.argv[2]
    start, checked = parse_time(sys.argv[3]), datetime.now(timezone.utc)
    if main.stem != session_id or not main.is_file() or start > checked:
        raise ValueError("현재 세션 경로/ID 또는 시작 시각 확인 필요")
    usage, models = read(main, session_id, start)
    if not usage:
        raise ValueError("이번 실행의 usage 기록 없음")
    main_responses, subagents = len(usage), 0
    for path in sorted(main.with_suffix("").glob("subagents/*.jsonl")):
        u, ms = read(path, session_id, start)
        if u:
            subagents += 1
            usage.update({(str(path), key): value for key, value in u.items()})
            models |= ms
    keys = ["input_tokens", "output_tokens", "cache_creation_input_tokens", "cache_read_input_tokens"]
    result = {key: sum(u.get(key) or 0 for u in usage.values()) for key in keys}
    result["total_input_tokens"] = sum(result[key] for key in keys if key != "output_tokens")
    result.update(responses=len(usage), main_responses=main_responses, subagents=subagents,
                  models=sorted(models), file=str(main), session_id=session_id,
                  started_at=start.astimezone(KST).isoformat(timespec="seconds"),
                  checked_at=checked.astimezone(KST).isoformat(timespec="seconds"),
                  elapsed_sec=int((checked - start).total_seconds()))
    print(json.dumps(result, ensure_ascii=False))
except Exception as error:
    print(json.dumps({"error": f"{type(error).__name__}: {error}"}, ensure_ascii=False))
EOF
```

숫자는 천 단위 쉼표를 넣고 `elapsed_sec`을 분·초로 바꾼다. 서브 응답 수는 `responses - main_responses`, 누적 입력은 일반 입력(비캐시)+캐시 쓰기+캐시 읽기다. 이는 여러 요청에 걸친 누적 처리량이며 청구 금액이 아니고, 집계 뒤 마지막 몇 턴은 포함되지 않는다. 명령이 실패하면 해당 줄에 `집계 불가 (오류 내용)`을 쓴다. 예정 시각은 스케줄러에서 확인한 경우만 기록하고, 예정 시각과 실제 시작·첫 게시가 모두 확인될 때만 지연 시간을 계산하며 지연 원인은 추정하지 않는다. 전송 시각은 성공 응답 뒤 상태 파일에 갱신한다.

```
⚙️ 실행 정보
• 조사: 한국 [시장 상태] · 미국 [시장 상태]
• 한국: 발견 n · 선정 n(신뢰도 낮음 n) · 적격 미선정 n · 제외 n · 보류 n · 미조사 n
• 미국: 발견 n · 선정 n(신뢰도 낮음 n) · 적격 미선정 n · 제외 n · 보류 n · 미조사 n
• 주요 사유: 시장별 최대 2개와 건수 또는 탐색 완료·발견 0개
• 토큰: 일반 입력 12,345 · 캐시 쓰기 123,456 · 캐시 읽기 1,234,567 · 누적 입력 1,370,368 · 출력 34,567
• 응답: 62회 (메인 62 · 서브에이전트 0개 0)
• 시간: 기준 07:00 · 예정 미확인 · 시작 07:04:00 · 첫 게시 07:10:00 · 집계 07:10:37 KST (소요 6분 37초)
• 기록: [실행 ID] · 상태 저장 성공 · 실행 간 보존 [확인/미확인]
• 모델: [현재 기록에서 확인한 모델명 또는 집계 불가]
```

## 실행·게시 순서

1. **기준 설정**: 실제 시작 시각·실행 ID·작업 환경·지침 해시를 기록하고 브리핑 날짜와 각 기간을 고정한다.
2. **채널 확인**: ID `C0CDFHG8PL7`로 읽고, 실패하면 `slack_search_channels`로 찾는다. 최근 7일의 부모를 모두 확인할 때까지 페이지를 읽어 중복 목록을 만든다. 오늘 게시물이 완성됐으면 종료하고, 오늘 부모가 있으면 재실행 규칙으로 누락분만 처리한다. 작업이 필요하면 상태 경로와 잠금을 확인한 뒤 채널을 다시 읽는다.
3. **탐색·빠른 확인**: 시장 탐색과 기사 탐색으로 후보를 모으고, 전체 후보에 시세 조회와 가치·실적 지표 조회를 돌려 저평가 1~3단계를 판정한다.
4. **상세 검증·확정**: 조사 순서대로 신호·저평가·실적 지속성·위험·목표가를 확인하고 `levels()`로 가격을 계산한 뒤 적격 기업의 순위를 정한다. 신뢰도 정상 기업이 없는 시장은 신뢰도 낮음 후보로 채운다.
5. **작성·검증**: 시장별 집계와 부모·답글을 작성하고, 아래 점검을 통과한 스냅샷을 저장·다시 읽어 확인한다. 선정 0개도 같다.
6. **게시**: 전송 직전 오늘 부모를 다시 확인한다. 다른 실행이 게시했다면 새 부모를 보내지 않고 그 부모와 일치하는 상태만 복원한다. 없으면 `slack_send_message`로 부모를 보내고 응답의 `ts`를 `thread_ts`로 써서 한국·미국 순으로 답글을 보낸다. 성공 시각·`ts`·시도 횟수를 즉시 저장한다. 초안 도구는 쓰지 않으며 `reply_broadcast`·`unfurl_app_links`는 설정하지 않는다.
7. **마무리**: 사용량을 집계해 실행 정보 답글을 보내고 게시 결과를 갱신한다. 실패·예외로 끝나도 실행 기록과 오류를 보존하고 자신이 소유한 잠금만 해제한다.

### 게시 전 점검

게시한 메시지는 수정하지 않으므로 틀린 곳은 전송 전에 고친다.

- **적격성**: 각 선정 기업이 여섯 필수 조건(신뢰도 낮음은 추세 제외)과 급등·상승여력 조건을 통과했고, 등급·가격상태·신호 점수·순위가 기록과 일치하는가. 신뢰도 낮음 기업은 그 시장에 신뢰도 정상 적격 기업이 없을 때만 선정됐고 라벨·미충족 조건·수익실현·손절 미산출 표시가 있는가.
- **내용**: 설명 1줄·포인트 3줄·추세 줄의 순서, 저평가·실적 근거의 출처 링크와 조회일·발행일, 모든 사실·전망 인용의 근거가 있고 권유·예측 문장이 없는가. 한국 공시 제목 신호는 후보별 완전 조회·후속 변경 점검을 통과했고 DART 접수일·링크·확인 방식이 있으며 미확인 세부 수치를 채우지 않았는가. 소송·규제는 당사자·내용·진행 단계·영향에 따라 판정했고 보도량을 순위에 쓰지 않았는가.
- **가격**: 종가 거래일이 `expected_date`와 같고, EPS/BPS·비교배수의 기간·회계·통화·단위·출처가 맞는가. `entry ≤ buy_ceiling`, `entry ≤ close`, `stop < entry < take_profit`이며 상승여력은 종가, 손익 비율은 진입가 기준인가.
- **추세**: `trend.date == expected_date`이고 연속 80세션의 동일 수정주가 계열로 판정했는가.
- **일치·표시**: 부모·답글·스냅샷의 회사명·순서·상승여력·가격상태·신뢰도·집계가 같은가. 부모 `검증:` 줄, 빈 시장 답글, 실행 정보의 시장 상태·건수·주요 사유(사유명·건수·순서)가 집계 객체 하나에서 코드로 만들어져 서로 같은가. 시가총액은 `display` 그대로이고, 링크·Markdown이 유효하며 메시지 길이를 코드로 확인했는가.
- **진단**: 탐색 여섯 종류와 발견 후보가 기록됐고 최종 상태 합계가 맞으며, 보류·조회 오류·미조사를 미충족과 구별했는가. 미조사는 조사 한도(그룹 번호)·실행 제한(빠른 확인/상세 검증)으로 나눠 셌고, 빠른 확인 미완료가 있으면 원인을 기록했는가. 시각·지침 해시·상태 경로·보존 여부를 기록하고 잠금·게시 직전 중복 확인을 했는가.

### 실패·재시도

- 실패한 메시지만 재시도한다. 먼저 채널·스레드를 다시 읽어 실제 게시 여부를 확인하고, 이미 있으면 다시 보내지 않는다.
- 채널을 찾지 못하거나 부모가 3번 실패하면 종료한다. 부모가 없으면 답글도 보내지 않는다.
- 답글 하나가 3번 실패하면 건너뛰고 나머지를 계속한다.

## 최종 결과

아래 상세 기록은 루틴의 마지막 응답과 실행별 조사 파일에 남기며 Slack에는 올리지 않는다. 게시 완료 여부와 조사 완료 여부는 구분한다. 각 항목의 세부는 **게시 전 스냅샷**의 저장 항목을 따른다.

```
상태: 완료 / 일부 게시 / 게시 안 함 / 실패
조사: 한국 [시장 상태] · 미국 [시장 상태]
부모 메시지: Slack 링크 또는 없음
답글: 한국 n개 기업(이름) · 미국 n개 기업(이름) · 실행 정보 게시 여부
실행 식별: 실행 ID · 세션 ID/미확인 · 작업 경로 · Git 커밋 · 지침 SHA-256
시간: 기준 시각 · 예정 시각/미확인 · 실제 시작 · 첫/최종 게시 · 집계 시각
상태 보존: 절대 경로 · 저장/다시 읽기 성공 여부 · 실행 간 보존 확인/미확인 · 잠금 처리

조사 범위와 집계: 시장별 탐색의 검색어·기간·결과·재시도·오류, 상태별·단계별·대표 사유별 집계와 합계 검증, 미조사 기업
조회 오류: 유형(출처·예외 이름)별 건수와 원문 예시, 재조회(`retry`)·추가 재실행 결과, 대체 출처·매매가 미산출 기업
후보·선정 근거: 모든 후보의 최종 상태·최초 차단 단계·사유·미수행 단계, 선정·적격 미선정 기업의 등급·신호 점수·순위와 저평가·진입가·실적·추세·신호 근거
위험 점검: 후보별 네 묶음 검색어·기간·판정 또는 미수행 사유, 발견 기사의 URL·발행일·사건 요약
오류: 실패한 메시지·시도 횟수·오류, 스냅샷 저장·복원 오류, 없으면 없음
```
