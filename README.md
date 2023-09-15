# 매출 하락 원인 분석

* 사용 데이터: Google Bigquery Demo Data인 Google Merchandise Store의 사용자 행동 데이터 사용.
* 데이터 기간: 2020년 11월 1일 ~ 2021년 1월 31일 3개월의 데이터 존재.
* 데이터 형태: GA4가 수집하는 형태로 이벤트를 기반으로 Struct 구조 데이터.
* 분석 환경: Google Bigquery 및 Google Data Studio 사용.

## 문제 정의

<image src = "https://github.com/wbin0718/google_analytics_dashboard/assets/104637982/ace29fa5-4564-45d8-9a61-e5322f26d646">

* 위 그래프를 보면 1월달의 매출이 11월과 12월과 비교했을 때 50% 수준의 매출을 보이고 있음.
* 1월의 매출이 급격히 감소하는 문제를 발견.

## 가설 설정

*  데이터에 포함되어 있는 (not set) 데이터로 인해 매출이 감소했을 것이다.
*  거래 건수가 감소했을 것이다.
* 웹페이지를 접속하는 사용자가 감소했을 것이다.

> 총 3개의 가설을 기준으로 분석을 진행

### not set 데이터로 인한 매출 감소

* GA4로 수집된 데이터를 확인하면 특정 user가 구매한 상품명과 상품 가격이 (not set) 처리된 경우가 448개가 있음.
* 매출로 집계가 될 때 (not set)으로 수집된 상품들은 포함되지 않는 문제가 있음.
* 따라서 단순히 매출이 감소한 것이 아니라 제대로 데이터가 수집되지 않아, 매출이 줄어든 것처럼 보이는 것일 수가 있음.

<image src = "https://github.com/wbin0718/google_analytics_dashboard/assets/104637982/9f0f3f8b-21e9-4b3c-989e-def20f716f6c">

* 위 그래프는 purchase 이벤트가 발생한 유저들을 월별로 나누고, 구매한 상품명과 상품 가격이 (not set)인 것을 집계한 그래프
* 1월의 (not set) 개수가 11월, 12월 보다 대략 250개 정도 많이 포함되어 있음.
* 250개가 더 많은 (not set)이 실제로 제대로 수집되었다면 11월, 12월과 비슷한 매출을 보였을까?
* 최소, 사용자가 상품을 몇 개씩 샀는지에 대한 데이터가 기록된다면, 유추할 수는 있지만, 이 또한 (not set)으로 전부 표시되어 정확히 알 수 없음.
* 하지만, 구매된 상품 중 가장 비싸게 판매된 상품의 가격은 120$임.
* 250 * 120 = 30000 으로 이를 1월 매출과 합해도 11월, 12월 매출에 비해서는 낮은 수준을 보임.
* 낮은 수준을 보이지만, 한 사용자가 120$ 상품을 여러개 샀을 수도 있으며, 여러개 샀을 때 꼭 120$의 상품을 사지 않았을 수도 있음.
* 즉, 가장 비싼 물건을 사용자가 하나씩 샀다는 가정이면, (not set)과는 상관없이 1월의 매출이 감소했다고 볼 수 있음.
* 하지만, 모든 조건을 현재 데이터로 파악할 수 없고, 매출 분석의 원인을 찾아가는 프로젝트를 진행하는 것이므로 (not set)의 영향이 없다고 가정하고 분석을 진행.
* 따라서 purchase 이벤트가 발생한 사용자 중 구매한 상품명, 가격, 개수 등이 (not set)인 데이터는 모두 제거하고 분석 진행.

### 거래 건수 감소로 인한 매출 감소

* 매출이 감소한 이유를 가장 먼저 상품을 구매하는 거래 건수가 줄었기 때문이라고 생각함.
* 당연한 듯, 한 사용자가 여러번 접속해서 구매를 하면 당연히 매출이 상승할 것임.
