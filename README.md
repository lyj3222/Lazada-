# Lazada 데이터 관리자 대시보드 with Text-to-SQL AI 챗봇

## 프로젝트 소개

본 프로젝트는 Lazada 상품 데이터를 활용하여 SQLite 데이터베이스를 구축하고, SQL 기반 데이터 분석 및 시각화를 수행하는 관리자 대시보드를 구현한 프로젝트입니다.

또한 OpenAI API를 활용한 Text-to-SQL AI 챗봇을 구현하여 사용자가 자연어로 질문하면 SQL을 자동 생성하고 결과를 조회할 수 있도록 개발하였습니다.

CSV 데이터 정제부터 데이터베이스 설계, SQL 분석, Gradio UI 구현, AI 챗봇까지 하나의 프로젝트로 통합하였습니다.

---

# 사용 기술

* Python
* Pandas
* SQLite3
* SQL
* Plotly
* Gradio
* OpenAI API

---

# 데이터 정제 전후 비교

### 정제 전

* 원본 데이터 : 1000건
* 결측치 존재
* final_price 100 미만 데이터 존재
* breadcrumb 컬럼이 JSON 문자열 형태
* rating = 0 데이터 존재
* No Brand / No brands 데이터 존재

### 정제 후

* 데이터 : 591건
* final_price 100 미만 데이터 제거
* 텍스트 결측치는 "없음", 숫자 결측치는 0으로 처리
* rating = 0 → NULL 처리
* breadcrumb를 level1 / level2 / level3로 분리
* 동일 SKU의 No Brand 데이터를 새로운 브랜드명으로 변환

---

# ERD

```
                              ┌─────────────────────────┐
                              │        브랜드           │
                              ├─────────────────────────┤
                              │ PK brand_id            │
                              │ 브랜드명               │
                              └──────────┬─────────────┘
                                         │1
                                         │
                                         │N
┌─────────────────────────┐      ┌───────▼──────────────────────────────────────────────┐
│        판매자           │      │                     상품(Product)                     │
├─────────────────────────┤      ├──────────────────────────────────────────────────────┤
│ PK seller_id           │      │ PK product_id                                         │
│ 판매자명               │      │ SKU                                                   │
│ 판매자 평점            │      │ 상품명                                                │
│ Super Seller 여부      │      │ 초기가격                                              │
│ LazMall 여부           │      │ 최종가격                                              │
└──────────┬─────────────┘      │ 할인율                                                │
           │1                   │ 평점                                                  │
           │                    │ 리뷰수                                                │
           │N                   │ 판매수량                                              │
           │                    │ GMV                                                  │
           │                    │ 재고                                                  │
           │                    │ 브랜드ID (FK)                                        │
           │                    │ 판매자ID (FK)                                        │
           │                    │ 카테고리ID (FK)                                      │
           │                    └───────────────────────┬──────────────────────────────┘
           │                                            │
           │                                            │N
           │                                            │
           │                                            │1
                                   ┌────────────────────▼────────────────────┐
                                   │             카테고리                    │
                                   ├─────────────────────────────────────────┤
                                   │ PK category_id                         │
                                   │ level1                                 │
                                   │ level2                                 │
                                   │ level3                                 │
                                   └─────────────────────────────────────────┘
```

### 테이블 구성

### Products

product_id (PK)

sku

title

initial_price

final_price

discount

rating

reviews

number_sold

gmv

stock

brand_id (FK)

seller_id (FK)

category_id (FK)

### Sellers

seller_id (PK)

seller_name

seller_rating

is_super_seller

is_lazmall

### Brands

brand_id (PK)

brand_name

### Categories

category_id (PK)

level1

level2

level3

---

# 구현 기능

* CSV 데이터 정제
* SQLite 데이터베이스 구축
* ERD 설계
* SQL 기반 데이터 분석
* Plotly 시각화
* Gradio 관리자 대시보드
* OpenAI Text-to-SQL AI 챗봇
* SQL 가드레일 구현

---

# 구현한 차트

1. KPI 카드

   * 총 상품 수
   * 총 셀러 수
   * 총 GMV
   * 평균 평점

2. 카테고리별 매출 TOP 10

3. 슈퍼셀러 vs 일반셀러 비교

4. 평점 분포 히스토그램

5. 베스트셀러 TOP 10

---

# AI 챗봇 테스트

| 질문                            | 결과    |
| ----------------------------- | ----- |
|정시배송률 95% 이상인 슈퍼셀러가 파는 상품 중 판매량이 높은 상품 10개 삭제해줘                | 정상 출력 |
|정시배송률 95% 이상인 슈퍼셀러가 파는 상품을 카테고리별 매출로 집계해줘        | 정상 출력 |
|슈퍼셀러가 아닌 판매자 중에서 정시 배송률이 90% 이하인 셀러들 찾아줘      | 정상 출력 |

※ 실행 화면은 screenshots 폴더를 참고

---

# 어려웠던 점 및 해결 방법

### 1. breadcrumb JSON 파싱

문제

* breadcrumb가 일반 문자열이 아닌 JSON 배열 형태로 저장되어 있어 카테고리 분석이 어려웠다.

해결

* json.loads()를 이용하여 level1, level2, level3 컬럼으로 분리하였다.

### 2. 브랜드 데이터 정제

문제

* No Brand 데이터가 많아 브랜드 분석의 정확도가 떨어졌다.

해결

* 동일 SKU를 기준으로 새로운 브랜드명을 생성하여 브랜드 마스터 테이블을 구성하였다.

### 3. SQLite 외래키(Foreign Key)

문제

* 상품 테이블에 Seller, Brand, Category를 연결하는 과정에서 ID 매핑이 필요하였다.

해결

* Pandas merge()를 이용하여 이름을 ID로 변환한 후 Products 테이블에 저장하였다.

### 4. AI Text-to-SQL

문제

* LLM이 잘못된 SQL을 생성하거나 위험한 SQL을 생성할 가능성이 있었다.

해결

* SELECT만 허용하는 SQL 가드레일을 구현하고 DROP, DELETE, UPDATE 등의 명령어를 차단하였다.

---
