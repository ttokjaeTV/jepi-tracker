# JEPI 분배 트래커

JEPI(JPMorgan Equity Premium Income ETF) 월 분배 이력, 연간 분배·성장률, 원화 환산, 구성종목(ELN 비중 따로), 국내상장 S&P500 커버드콜 ETF 3종 비교를 한 페이지로 보여 줍니다.
GitHub Actions가 매일 새 데이터를 확인해 자동으로 갱신합니다. 구조·화면은 `jepq-tracker`(원형 `schd-tracker`)와 같습니다.

## 폴더 구조

| 경로 | 내용 |
|---|---|
| `index.html` | 트래커 화면. `data/*.json`을 읽어서 그립니다 |
| `data/dividends.json` | JEPI 분배 (배당락일·기준일·지급일·금액·배당락일 종가·지급일 환율·출처). 2020-07 첫 분배부터 전체 |
| `data/holdings.json` | 구성종목 전체 목록(주식·ELN·현금). `asOf` = JPMorgan 공시 기준일 |
| `data/holdings_monthly.json` | 월별 비중 기록: 매월 첫 JPMorgan 공식 파일(2026-10부터, `basis: official`) + SEC N-PORT 분기말 공시(2020-06~, `basis: nport`, `accession` = 공시 번호). `isin` = N-PORT ISIN → 티커 (한 달 = 한 줄, ELN 합계 포함) |
| `data/kr_etf.json` | 국내 JEPI형(S&P500 커버드콜) ETF 3종 분배 이력, 총보수, 실부담비용 |
| `data/prices.json` | JEPI 일별 종가·수정주가 (2020-05-21 첫 거래일 이후 전체) |
| `data/tr_compare.json` | 배당 재투자 총수익(TR) 비교 차트용: JEPI·JEPQ·QQQ·SCHD·SPY 일별 지수 (JEPQ 첫 거래일 2022-05-04 = 100). SCHD·JEPQ·JEPI 트래커 세 곳이 같은 파일 |
| `data/meta.json` | 갱신일, 한글 종목명·섹터명, 원천징수율, 보수(`jepi_fee`), 공식 수익률(`official`), 분배 일정(`schedule`) |
| `downloads/jepi_dividend_tracker.xlsx` | 엑셀 마스터 (데이터가 바뀔 때마다 자동 생성) |
| `scripts/update.py` | 데이터 수집·검증 |
| `scripts/holdings_lib.py` | 구성종목 공통 처리 (JPMorgan 보유종목 엑셀 해석·SEC N-PORT XML 해석(`parse_nport`, `python scripts/holdings_lib.py`로 자체 점검)·월별 기록) |
| `scripts/build_xlsx.py` | 엑셀 생성 |
| `scripts/tr_compare.py` | TR 비교 수집·요약 계산 (세 트래커 같은 파일, 직접 실행하면 계산 점검) |
| `.github/workflows/update.yml` | 매일 한국시간 08:30 자동 실행 |

## 출처와 접속 방법 (2026-11-01 실측)

| 자료 | 출처 (우선순위 순) | 접속 방법 | 실측 결과 |
|---|---|---|---|
| 최신 분배 1건 | JPMorgan 상품 데이터 `FundsMarketingHandler/product-data?cusip=46641Q332` | 일반 `requests` (브라우저 UA) | 성공. 배당락·기준·지급일·금액(소수 5자리) |
| 과거 분배 | stockanalysis 분배 API `api/symbol/e/jepi/dividend` | 일반 `requests` | 성공. 2021-10 이후 61건, 소수 5자리(가끔 3자리 표기), 기준일·지급일 포함 |
| 〃 예비·초기분 | wisesheets `etf/JEPI/dividends` (페이지 안 JSON) | 일반 `requests` | 성공. 2020-07 첫 분배부터 전체, 소수 5자리. 2020-07~2021-09 15건은 이 출처 값 |
| 〃 교차 확인 | dividendhistory.net·dividendvision·Yahoo Finance·digrin | 일반 요청 | wisesheets와 일치(Yahoo는 3자리, digrin은 4자리 반올림) |
| Nasdaq.com 분배 API | `api.nasdaq.com/api/quote/JEPI/dividends` | — | **실패.** JEPI는 NYSE Arca 상장이라 "Non-Nasdaq symbols" 응답 → 주가에만 사용 |
| JPMorgan 과거 분배 이력 | — | — | 공식 이력 파일 없음 |
| 주가·수정주가 | Yahoo Finance → Nasdaq.com 종가 + 분배 재투자 직접 계산 | `yfinance` / `requests` | 성공 (PC에서는 사용자 폴더 경로의 한글 때문에 `yfinance` 인증서 오류 → GitHub 서버에서는 문제 없음. 첫 채우기는 Yahoo 차트 API를 curl로 받음) |
| 구성종목 | JPMorgan 일별 보유종목 엑셀 `FundsMarketingHandler/excel?type=dailyETFHoldings` | 일반 `requests` | 성공. 주식 117 + ELN 15 + 단기자금·현금 |
| 과거 분기말 구성종목 | SEC EDGAR Form N-PORT (CIK 1485894, 시리즈 S000068402). 목록은 `efts.sec.gov/LATEST/search-index`(NPORT-P·NPORT-P/A 따로 조회, 같은 분기는 가장 늦게 낸 정정본), 본문은 `Archives/edgar/data/1485894/<공시번호>/primary_doc.xml`. 티커는 OpenFIGI(키 없이) ISIN 조회 | `requests`, User-Agent `ttokjaeTV research ttokjae@gmail.com`, 요청 사이 0.2초 | 성공(2026-11-02). 분기말만 공개, 약 60일 뒤 |
| 공식 수익률 | JPMorgan 상품 데이터 (30일 SEC·12개월 롤링 분배수익률·분배수익률·NAV·순자산·보수) | 일반 `requests` | 성공 |
| 분배 일정 | JPMorgan 'ETF Dividend Calendar' PDF (연 1회) | 일반 `requests` + `pypdf` | 성공. 2026년판(JEPI = Monthly Group 1, JEPQ와 같은 일정), 2027년판은 아직 미공개 |
| 지급일 환율 | 하나은행 매매기준율(다음금융) | 일반 `requests` | 성공. 지급일이 지나야 채워짐 |
| 국내 ETF 분배금 | FunETF 분배 이력 API | 일반 `requests` | 성공 |
| 국내 ETF 실부담비용 | 금융투자협회 전자공시 '펀드별 보수비용비교' | 일반 `requests` (XML 서비스) | 성공. 2026-09-30 기준 |

## 자동 갱신 규칙

- **JEPI 분배**: stockanalysis 이력(막히면 wisesheets)으로 빠진 달을 채우고, 가장 최근 1건은 JPMorgan 공식 값으로 덮어씁니다. 새 분배가 직전 대비 ±60% 넘게 다르면 반영하지 않고 이슈로 알림
  - ±60% 근거: 2020-07 ~ 2026-11 월간 변화 76회의 표준편차 18%, 최대 +56.5%(2020-09)·-51.6%(2021-02). 2023년 이후는 표준편차 13%, 최대 +38.1%(2024-09)·-29.7%. 그 1.6배로 잡아 상장 이후 실제 변동은 모두 통과하고 소수점 밀림·종목 혼동 같은 자료 오류만 거릅니다
  - 기존 기록과 출처 금액은 0.000015달러까지 같은 값으로 봅니다(stockanalysis가 2025-06 분배를 `$0.540`으로만 보여 주는 표기 차이 때문)
- **분배 월**: 지급일이 속한 달로 셉니다(배당락일은 보통 매월 첫 영업일, 지급은 2~3영업일 뒤). 2020-12-30 배당락분은 2021-01 지급이라 2020년은 6회입니다
- **배당락일 종가**: Yahoo Finance (배당 미조정 종가). 배당락일이 지나야 채워짐
- **주가 흐름**: 전체 이력을 7일마다(새 분배 반영 시엔 즉시) 통째로 다시 받음
- **TR 비교 차트**: Yahoo Finance 수정주가(Adj Close, 배당 재투자 반영) 5종을 7일마다 받아 `data/tr_compare.json`에 반영. 이미 있는 날짜는 그대로 두고 새 날짜만 덧붙임(Yahoo 소수점이 받을 때마다 조금씩 달라 과거 값이 흔들리지 않게). 못 받으면 기존 값 유지, 마지막 날짜가 14일 이상 오래되면 `확인 필요` 이슈. 화면의 1년·3년·전체 버튼은 그 구간 첫날을 0%로 다시 맞춘 누적 수익률이고, 엑셀 `TR비교` 시트의 수식 값과 같음. 배당 재투자 가정, 세금·환율 미반영
- **지급일 환율**: 하나은행 매매기준율(다음금융). 지급일이 지나야 채워짐
- **구성종목**: 매일 JPMorgan 공식 보유종목 엑셀을 받아 기준일이 새로우면 반영. ELN(주가연계채권, S&P500 콜옵션 매도 수익을 담는 상품, 기호 `SPX_숫자`)은 따로 묶어 비중을 보여 줌. 공식 파일을 10일 넘게 못 받으면 `확인 필요` 이슈
- **월별 비중 기록**: 그 달 첫 공식 파일을 `holdings_monthly.json`에 추가. JPMorgan은 과거 보유종목 파일을 공개하지 않아 2026-10부터 쌓입니다. JEPI는 정해진 지수 리밸런싱일이 없는 액티브 ETF입니다
- **N-PORT 분기말 기록**: 매주 월요일 SEC N-PORT 공시 목록을 확인해 새 분기말을 `basis: nport`로 추가(그 달에 공식 기록이 있으면 건너뜀). 구분: 주식(EC) / ELN(SN 중 제목에 ELN·linked to) / 현금(STIV). 증권대여 담보 MMF와 선물(평가손익만 있음)은 뺌. 비중은 공식 파일과 같게 시장가치 ÷ 포함한 줄 합계(100%)로 다시 계산(공시 `pctVal`은 순자산 대비라 합계가 100이 아님). N-PORT에는 섹터·티커가 없어 티커는 OpenFIGI, 섹터는 JPMorgan 공식 파일에 나온 종목만 붙이고 나머지는 '섹터 미확인'. OpenFIGI에 미국 줄이 없는 옛 ISIN은 `update.py`의 `FIGI_FIX`에 적음. 사이 달은 비워 둡니다(지어내지 않음). 수집에 실패하면 기존 값 유지, 최근 분기말 기록이 170일 넘게 새로워지지 않으면 `확인 필요` 이슈
- **공식 수익률**: 30일 SEC 수익률·12개월 롤링 분배수익률·분배수익률·NAV를 기준일과 함께 `data/meta.json`의 `official`에 저장. SEC 기준일이 7일 이상 새로워졌을 때만 갱신(매일 커밋 방지). 실패하면 기존 값 유지
- **다음 분배 예정일**: JPMorgan 'ETF Dividend Calendar' PDF에서 JEPI가 속한 그룹의 배당락·지급일을 읽어 `data/meta.json`의 `schedule`에 넣음. 올해·내년 PDF를 확인해 아직 없는 해만 읽습니다. 12월 자본이득 분배일은 `kind: capgain`으로 구분(자본이득이 없으면 지급되지 않을 수 있음)
- **국내 JEPI형 분배금**: FunETF 분배 이력
- **국내 JEPI형 실부담비용**: 금융투자협회 전자공시 '펀드별 보수비용비교'(총보수+기타비용+매매·중개수수료+판매수수료). 월말 기준·약 한 달 뒤 공개라, 지난달 말 기준이 공개되면 자동으로 바꿈(`data/kr_etf.json`의 `cost`)
- 새 분배가 반영되면 `JEPI 새 분배 반영` 이슈가, 수집 실패가 있으면 `JEPI 자동 갱신 확인 필요` 이슈가 생깁니다 (GitHub 알림 메일로 받음)

## 국내 비교 대상 (기초지수가 S&P500인 커버드콜)

| 코드 | 이름 | 고른 이유 |
|---|---|---|
| 482730 | TIGER 미국S&P500타겟데일리커버드콜 | 기초지수 S&P500 10% Daily Premium Covered Call Index(TR). 매일 옵션 매도, 연 10% 프리미엄 목표. 2024-05 상장으로 가장 오래 비교 가능. 2026-09부터 분배 기준일이 월말 → 월중(14일 무렵) |
| 0005A0 | KODEX 미국S&P500데일리커버드콜OTM | 기초지수 S&P 500 Covered Call 1% OTM Daily Index(TR). 매일 외가격 1% 옵션 매도. 기준일 월말 |
| 0138T0 | RISE 미국S&P500데일리고정커버드콜 | 기초지수 S&P500 10% Daily Fixed Covered Call Index. 매일 일정 비율 옵션 매도. 2025-12 상장, 2026-04부터 기준일 월중 |

제외: ACE 미국500데일리타겟커버드콜(480030, Bloomberg US 500 지수), SOL 미국500타겟데일리커버드콜액티브(494210, KEDI 미국퀄리티500), KODEX 미국S&P500배당귀족커버드콜(276970, 배당귀족 지수), KODEX 미국S&P500변동성확대시커버드콜(0089C0, 변동성이 커질 때만 옵션을 파는 구조라 매달 프리미엄을 받는 JEPI와 성격이 다름 — 필요하면 추가 가능).

## 국내 ETF 12개월 분배율 계산 (화면·엑셀 공통)

12개월 분배율 = 12개월 분배금 합계 ÷ 기준 가격. 회차별 분배율을 더한 값이 아닙니다.

- 기간: 비교 기준일(최신 분배월 말일) 직전 12개월에 분배 기준일이 든 회차 전부
- 기준 가격: 최근 분배 시점 1좌 가격. 국내 ETF는 `최근 분배금 ÷ 그 회차 분배율`로 역산(FunETF는 가격을 따로 주지 않음), JEPI는 최근 배당락일 종가
- JEPI 행은 기준일 이전 마지막 배당락까지 12회라, 위쪽 요약의 TTM(최신 분배 포함)과 기준일 이후 분배만큼 다를 수 있음 (예: 기준일 10-31 이전 마지막 배당락 10-01까지 $4.5637, 11-02 분배 포함 시 $4.5642)
- 화면은 `index.html`의 국내 비교표, 엑셀은 `[국내JEPI형_비교]` 시트(기준 가격·12개월 분배율 열)가 같은 식을 씁니다

## JEPQ 트래커와 다른 점

- 대상 지수 나스닥100 → S&P500. ELN 기호 `NDX_` → `SPX_`
- 과거 분배 출처: Nasdaq.com 분배 API가 JEPI를 지원하지 않아 stockanalysis(→ wisesheets)로 바꿈
- 상장 첫해 2022년 7회 → 2020년 6회. 변동 경고 범위 ±50% → ±60%
- 국내 비교 대상 3종 교체, 그중 2종은 기준일이 월중
- 엑셀 파일 이름 `downloads/jepi_dividend_tracker.xlsx`, 시트 `JEPI_배당`·`국내JEPI형_분배`·`국내JEPI형_비교`

## 손으로 고칠 때

- 값은 `data/*.json`에서 고칩니다. 엑셀은 다음 갱신 때 새로 만들어지므로 엑셀에서 고친 내용은 남지 않습니다.
- 분배 일정 PDF 형식이 바뀌어 자동으로 못 읽으면 `data/meta.json`의 `schedule.rows`에 `ex`·`record`·`pay`·`kind`(`income`/`capgain`)를 직접 넣고 `source`·`checked`를 고칩니다.
- 공식 수익률을 손으로 고칠 때는 `official`의 `value`(소수, 7.22% → 0.0722)와 `asOf`를 같이 고칩니다.
- 국내 ETF 총보수가 바뀌면 `data/kr_etf.json`의 `fee`와 `fee_source`를, 분배 기준일이 월말↔월중으로 바뀌면 `timing`을 고칩니다. 새 ETF를 추가하면 상장일 `listed`(YYYY-MM-DD)도 넣습니다.
- 한글 종목명이 없는 종목은 영문명으로 나옵니다. `data/meta.json`의 `names_ko`에 티커별로 추가합니다.
- Actions 탭 → `JEPI 데이터 자동 갱신` → `Run workflow`로 언제든 수동 실행할 수 있습니다.

## 로컬에서 보기

`index.html`을 더블클릭하면 데이터를 못 읽습니다(브라우저 보안). 폴더에서 `python -m http.server` 실행 후 http://localhost:8000 으로 엽니다.

출처: J.P. Morgan Asset Management, SEC EDGAR(Form N-PORT), OpenFIGI, stockanalysis, wisesheets, Nasdaq.com, Yahoo Finance, 다음금융(하나은행 고시 환율), FunETF, 금융투자협회. 특정 종목의 매수·매도를 권하는 자료가 아닙니다.
