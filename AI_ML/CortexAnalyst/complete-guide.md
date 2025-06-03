# Snowflake Cortex Analyst 시맨틱 모델 완전 가이드

## 📖 목차
- [개요](#개요)
- [1단계: 물리적 테이블 생성](#1단계-물리적-테이블-생성)
- [2단계: 시맨틱 모델의 주요 구성 요소](#2단계-시맨틱-모델의-주요-구성-요소)
- [3단계: 단계별 시맨틱 모델 작성](#3단계-단계별-시맨틱-모델-작성)
- [4단계: 각 구성 요소의 역할과 의미](#4단계-각-구성-요소의-역할과-의미)
- [5단계: 실제 사용 시나리오](#5단계-실제-사용-시나리오)
- [6단계: base_table 심화 이해](#6단계-base_table-심화-이해)
- [7단계: 다양한 base_table 설정 방법](#7단계-다양한-base_table-설정-방법)

## 개요

이 가이드는 Snowflake Cortex Analyst의 시맨틱 모델을 처음부터 차근차근 이해할 수 있도록 실제 물리적 테이블 예제를 통해 설명합니다.

## 1단계: 물리적 테이블 생성

온라인 쇼핑몰의 매출 데이터를 예로 들어 실제 테이블들을 만들어보겠습니다.

### 1.1 데이터베이스 및 스키마 생성

```sql
-- 데이터베이스와 스키마 생성
CREATE DATABASE ECOMMERCE_DB;
CREATE SCHEMA ECOMMERCE_DB.SALES;
CREATE SCHEMA ECOMMERCE_DB.MASTER;
```

### 1.2 물리적 테이블 생성

#### 주문 테이블 (ORDERS)
```sql
CREATE TABLE ECOMMERCE_DB.SALES.ORDERS (
    ORDER_ID VARCHAR(20) PRIMARY KEY,
    CUSTOMER_ID VARCHAR(20),
    ORDER_DATE DATE,
    ORDER_AMOUNT DECIMAL(10,2),
    SHIPPING_COST DECIMAL(8,2),
    DISCOUNT_AMOUNT DECIMAL(8,2),
    STATUS VARCHAR(20),
    PAYMENT_METHOD VARCHAR(30),
    SHIPPING_ADDRESS_STATE VARCHAR(50)
);
```

#### 주문 상세 테이블 (ORDER_ITEMS)
```sql
CREATE TABLE ECOMMERCE_DB.SALES.ORDER_ITEMS (
    ORDER_ITEM_ID VARCHAR(30) PRIMARY KEY,
    ORDER_ID VARCHAR(20),
    PRODUCT_ID VARCHAR(20),
    QUANTITY INTEGER,
    UNIT_PRICE DECIMAL(8,2),
    TOTAL_PRICE DECIMAL(10,2)
);
```

#### 제품 마스터 테이블 (PRODUCTS)
```sql
CREATE TABLE ECOMMERCE_DB.MASTER.PRODUCTS (
    PRODUCT_ID VARCHAR(20) PRIMARY KEY,
    PRODUCT_NAME VARCHAR(200),
    CATEGORY VARCHAR(50),
    BRAND VARCHAR(50),
    COST_PRICE DECIMAL(8,2),
    LIST_PRICE DECIMAL(8,2)
);
```

#### 고객 마스터 테이블 (CUSTOMERS)
```sql
CREATE TABLE ECOMMERCE_DB.MASTER.CUSTOMERS (
    CUSTOMER_ID VARCHAR(20) PRIMARY KEY,
    CUSTOMER_NAME VARCHAR(100),
    EMAIL VARCHAR(100),
    PHONE VARCHAR(20),
    REGISTRATION_DATE DATE,
    CUSTOMER_SEGMENT VARCHAR(30)
);
```

### 1.3 샘플 데이터 삽입

```sql
-- 제품 데이터
INSERT INTO ECOMMERCE_DB.MASTER.PRODUCTS VALUES
('P001', '아이폰 15 Pro', '전자제품', 'Apple', 800.00, 1200.00),
('P002', '삼성 갤럭시 S24', '전자제품', 'Samsung', 700.00, 1000.00),
('P003', '나이키 운동화', '신발', 'Nike', 60.00, 120.00),
('P004', '아디다스 티셔츠', '의류', 'Adidas', 20.00, 45.00),
('P005', '로지텍 마우스', '전자제품', 'Logitech', 25.00, 50.00),
('P006', '스타벅스 텀블러', '생활용품', 'Starbucks', 15.00, 30.00);

-- 고객 데이터
INSERT INTO ECOMMERCE_DB.MASTER.CUSTOMERS VALUES
('C001', '김철수', 'kim@email.com', '010-1234-5678', '2023-01-15', 'VIP'),
('C002', '이영희', 'lee@email.com', '010-2345-6789', '2023-03-20', '일반'),
('C003', '박민수', 'park@email.com', '010-3456-7890', '2023-06-10', '일반'),
('C004', '최지영', 'choi@email.com', '010-4567-8901', '2022-12-05', 'VIP');

-- 주문 데이터
INSERT INTO ECOMMERCE_DB.SALES.ORDERS VALUES
('ORD001', 'C001', '2024-01-15', 1200.00, 10.00, 0.00, '배송완료', '신용카드', '서울'),
('ORD002', 'C002', '2024-01-16', 165.00, 5.00, 10.00, '배송완료', '계좌이체', '부산'),
('ORD003', 'C003', '2024-01-17', 50.00, 5.00, 0.00, '주문취소', '신용카드', '대구'),
('ORD004', 'C001', '2024-01-18', 75.00, 5.00, 5.00, '배송완료', '신용카드', '서울'),
('ORD005', 'C004', '2024-01-19', 1000.00, 10.00, 50.00, '배송중', '계좌이체', '인천'),
('ORD006', 'C002', '2024-01-20', 30.00, 3.00, 0.00, '배송완료', '카카오페이', '부산');

-- 주문 상세 데이터
INSERT INTO ECOMMERCE_DB.SALES.ORDER_ITEMS VALUES
('OI001', 'ORD001', 'P001', 1, 1200.00, 1200.00),
('OI002', 'ORD002', 'P003', 1, 120.00, 120.00),
('OI003', 'ORD002', 'P004', 1, 45.00, 45.00),
('OI004', 'ORD003', 'P005', 1, 50.00, 50.00),
('OI005', 'ORD004', 'P004', 1, 45.00, 45.00),
('OI006', 'ORD004', 'P006', 1, 30.00, 30.00),
('OI007', 'ORD005', 'P002', 1, 1000.00, 1000.00),
('OI008', 'ORD006', 'P006', 1, 30.00, 30.00);
```

### 1.4 분석용 통합 뷰 생성

```sql
-- 분석을 위한 뷰 생성 (이 뷰가 시맨틱 모델의 기반이 됩니다)
CREATE VIEW ECOMMERCE_DB.SALES.SALES_SUMMARY AS
SELECT 
    o.ORDER_ID,
    o.ORDER_DATE,
    o.CUSTOMER_ID,
    c.CUSTOMER_NAME,
    c.CUSTOMER_SEGMENT,
    o.SHIPPING_ADDRESS_STATE as REGION,
    o.STATUS as ORDER_STATUS,
    o.PAYMENT_METHOD,
    oi.PRODUCT_ID,
    p.PRODUCT_NAME,
    p.CATEGORY as PRODUCT_CATEGORY,
    p.BRAND,
    oi.QUANTITY,
    oi.UNIT_PRICE,
    oi.TOTAL_PRICE as ITEM_REVENUE,
    p.COST_PRICE * oi.QUANTITY as ITEM_COST,
    (oi.TOTAL_PRICE - (p.COST_PRICE * oi.QUANTITY)) as ITEM_PROFIT,
    o.ORDER_AMOUNT,
    o.SHIPPING_COST,
    o.DISCOUNT_AMOUNT
FROM ECOMMERCE_DB.SALES.ORDERS o
JOIN ECOMMERCE_DB.SALES.ORDER_ITEMS oi ON o.ORDER_ID = oi.ORDER_ID
JOIN ECOMMERCE_DB.MASTER.PRODUCTS p ON oi.PRODUCT_ID = p.PRODUCT_ID
JOIN ECOMMERCE_DB.MASTER.CUSTOMERS c ON o.CUSTOMER_ID = c.CUSTOMER_ID;
```

### 1.5 테이블 구조 요약

| 테이블명 | 주요 컬럼 | 설명 |
|---------|-----------|------|
| **ORDERS** | ORDER_ID, CUSTOMER_ID, ORDER_DATE, ORDER_AMOUNT, STATUS | 주문 기본 정보 |
| **ORDER_ITEMS** | ORDER_ID, PRODUCT_ID, QUANTITY, UNIT_PRICE | 주문별 상품 상세 |
| **PRODUCTS** | PRODUCT_ID, PRODUCT_NAME, CATEGORY, BRAND, COST_PRICE | 상품 마스터 |
| **CUSTOMERS** | CUSTOMER_ID, CUSTOMER_NAME, CUSTOMER_SEGMENT | 고객 마스터 |
| **SALES_SUMMARY** | 모든 정보 통합 | 분석용 통합 뷰 |

## 2단계: 시맨틱 모델의 주요 구성 요소

시맨틱 모델은 **물리적 테이블**을 **비즈니스 사용자가 이해하기 쉬운 논리적 개념**으로 변환합니다.

### 2.1 주요 구성 요소

| 구성 요소 | 설명 | 예시 |
|-----------|------|------|
| **Dimensions (차원)** | "무엇을", "누가", "어디서" 같은 분류 정보 | 제품 카테고리, 고객 세그먼트, 지역, 브랜드 |
| **Time Dimensions (시간 차원)** | "언제" 관련 정보 | 주문일, 등록일 |
| **Measures/Facts (측정값)** | "얼마나" 같은 수치 정보 | 매출액, 수량, 이익 |
| **Relationships (관계)** | 테이블 간 연결 방법 | 주문 ↔ 고객, 주문 ↔ 상품 |

### 2.2 물리적 vs 논리적 테이블

#### 물리적 테이블 (Physical Tables)
실제 Snowflake 데이터베이스에 저장된 테이블들
```
ECOMMERCE_DB.SALES.ORDERS          ← 실제 테이블
ECOMMERCE_DB.SALES.ORDER_ITEMS     ← 실제 테이블  
ECOMMERCE_DB.MASTER.PRODUCTS       ← 실제 테이블
ECOMMERCE_DB.MASTER.CUSTOMERS      ← 실제 테이블
ECOMMERCE_DB.SALES.SALES_SUMMARY   ← 실제 뷰 (여러 테이블 조인)
```

#### 논리적 테이블 (Logical Tables)
시맨틱 모델에서 정의하는 비즈니스 관점의 테이블
```yaml
tables:
  - name: sales_data              ← 논리적 테이블명 (사용자가 이해하기 쉬운 이름)
    base_table:                   ← 실제 물리적 테이블/뷰 지정
      database: ECOMMERCE_DB
      schema: SALES  
      table: SALES_SUMMARY        ← 여기가 핵심!
```

## 3단계: 단계별 시맨틱 모델 작성

### 3.1 완전한 시맨틱 모델 YAML

```yaml
# =============================================================================
# STEP 1: 기본 정보 (모델의 이름과 설명)
# =============================================================================
name: ecommerce_sales_analysis
description: "온라인 쇼핑몰 매출 분석을 위한 시맨틱 모델"
comments: "주문, 고객, 상품 데이터를 통합하여 매출 분석 제공"

# =============================================================================
# STEP 2: 논리적 테이블 정의 (물리적 테이블을 논리적으로 표현)
# =============================================================================
tables:
  # 첫 번째 논리적 테이블: 매출 분석의 중심이 되는 테이블
  - name: sales_data
    description: "주문별 매출 및 상품 정보를 통합한 분석 테이블"
    
    # 실제 물리적 테이블/뷰 지정
    base_table:
      database: ECOMMERCE_DB
      schema: SALES
      table: SALES_SUMMARY  # 앞에서 만든 뷰 사용
    
    # =========================================================================
    # STEP 3: Dimensions (차원) - 분류/카테고리 정보
    # =========================================================================
    dimensions:
      # 상품 관련 차원들
      - name: product_category
        synonyms: ["제품군", "상품 카테고리", "카테고리", "제품 분류"]
        description: "상품이 속한 카테고리 (전자제품, 신발, 의류 등)"
        expr: PRODUCT_CATEGORY  # 실제 컬럼명
        data_type: varchar
        unique: false
        sample_values:
          - "전자제품"
          - "신발" 
          - "의류"
          - "생활용품"
      
      - name: brand
        synonyms: ["브랜드", "제조사", "메이커"]
        description: "상품 브랜드명"
        expr: BRAND
        data_type: varchar
        unique: false
        sample_values:
          - "Apple"
          - "Samsung"
          - "Nike"
          - "Adidas"
      
      # 고객 관련 차원들  
      - name: customer_segment
        synonyms: ["고객등급", "고객 세그먼트", "회원등급"]
        description: "고객의 등급 분류"
        expr: CUSTOMER_SEGMENT
        data_type: varchar
        unique: false
        sample_values:
          - "VIP"
          - "일반"
      
      # 지역 관련 차원
      - name: region
        synonyms: ["지역", "배송지역", "시도"]
        description: "주문 배송 지역"
        expr: REGION
        data_type: varchar
        unique: false
        sample_values:
          - "서울"
          - "부산"
          - "대구"
          - "인천"
      
      # 주문 상태 차원
      - name: order_status
        synonyms: ["주문상태", "배송상태", "주문 상황"]
        description: "주문의 현재 상태"
        expr: ORDER_STATUS
        data_type: varchar
        unique: false
        sample_values:
          - "배송완료"
          - "배송중"
          - "주문취소"
      
      # 결제 방법 차원
      - name: payment_method
        synonyms: ["결제방법", "결제수단", "페이먼트"]
        description: "고객이 사용한 결제 방법"
        expr: PAYMENT_METHOD
        data_type: varchar
        unique: false
        sample_values:
          - "신용카드"
          - "계좌이체"
          - "카카오페이"
    
    # =========================================================================
    # STEP 4: Time Dimensions (시간 차원) - 날짜/시간 정보
    # =========================================================================
    time_dimensions:
      - name: order_date
        synonyms: ["주문일", "주문날짜", "구매일", "구매날짜"]
        description: "고객이 주문한 날짜"
        expr: ORDER_DATE
        data_type: date
        unique: false
    
    # =========================================================================
    # STEP 5: Measures (측정값) - 수치 데이터
    # =========================================================================
    measures:
      # 매출 관련 측정값들
      - name: total_revenue
        synonyms: ["총매출", "매출액", "총 매출액", "매출"]
        description: "상품별 총 매출액 (할인 전)"
        expr: ITEM_REVENUE
        data_type: number
        default_aggregation: sum
      
      - name: total_cost
        synonyms: ["총원가", "원가", "비용"]
        description: "상품별 총 원가"
        expr: ITEM_COST
        data_type: number
        default_aggregation: sum
      
      - name: profit
        synonyms: ["이익", "수익", "총이익", "순이익"]
        description: "상품별 이익 (매출 - 원가)"
        expr: ITEM_PROFIT
        data_type: number
        default_aggregation: sum
      
      - name: quantity_sold
        synonyms: ["판매수량", "수량", "판매량", "개수"]
        description: "판매된 상품 수량"
        expr: QUANTITY
        data_type: number
        default_aggregation: sum
      
      - name: unit_price
        synonyms: ["단가", "개당가격", "상품가격"]
        description: "상품의 개당 판매 가격"
        expr: UNIT_PRICE
        data_type: number
        default_aggregation: avg
      
      - name: order_count
        synonyms: ["주문수", "주문건수", "거래수"]
        description: "총 주문 건수"
        expr: ORDER_ID
        data_type: varchar
        default_aggregation: count_distinct
      
      - name: shipping_cost
        synonyms: ["배송비", "배송료", "배송 비용"]
        description: "주문별 배송비"
        expr: SHIPPING_COST
        data_type: number
        default_aggregation: sum
      
      - name: discount_amount
        synonyms: ["할인금액", "할인액", "디스카운트"]
        description: "주문에 적용된 할인 금액"
        expr: DISCOUNT_AMOUNT
        data_type: number
        default_aggregation: sum

# =============================================================================
# STEP 6: 검증된 쿼리 (Verified Queries) - 자주 묻는 질문과 답
# =============================================================================
verified_queries:
  - question: "2024년 1월 총 매출은 얼마인가요?"
    sql: |
      SELECT SUM(ITEM_REVENUE) as total_revenue
      FROM ECOMMERCE_DB.SALES.SALES_SUMMARY
      WHERE DATE_TRUNC('MONTH', ORDER_DATE) = '2024-01-01'
      AND ORDER_STATUS != '주문취소'
    verified_at: "2024-01-25"
  
  - question: "제품 카테고리별 매출 현황을 보여주세요"
    sql: |
      SELECT 
        PRODUCT_CATEGORY,
        SUM(ITEM_REVENUE) as total_revenue,
        SUM(QUANTITY) as total_quantity,
        COUNT(DISTINCT ORDER_ID) as order_count
      FROM ECOMMERCE_DB.SALES.SALES_SUMMARY
      WHERE ORDER_STATUS != '주문취소'
      GROUP BY PRODUCT_CATEGORY
      ORDER BY total_revenue DESC
    verified_at: "2024-01-25"
  
  - question: "VIP 고객들의 평균 주문 금액은?"
    sql: |
      SELECT AVG(ORDER_AMOUNT) as avg_order_amount
      FROM ECOMMERCE_DB.SALES.ORDERS o
      JOIN ECOMMERCE_DB.MASTER.CUSTOMERS c ON o.CUSTOMER_ID = c.CUSTOMER_ID
      WHERE c.CUSTOMER_SEGMENT = 'VIP'
      AND o.STATUS != '주문취소'
    verified_at: "2024-01-25"

# =============================================================================
# STEP 7: 필터 조건 (Filters) - 자주 사용되는 필터링 조건
# =============================================================================
filters:
  - name: completed_orders_only
    description: "취소되지 않은 완료된 주문만 조회"
    expr: "ORDER_STATUS != '주문취소'"
  
  - name: current_year
    description: "올해 주문만 조회"
    expr: "YEAR(ORDER_DATE) = YEAR(CURRENT_DATE())"
  
  - name: vip_customers_only
    description: "VIP 고객만 조회"
    expr: "CUSTOMER_SEGMENT = 'VIP'"
```

## 4단계: 각 구성 요소의 역할과 의미

### 4.1 Dimensions (차원) - "분류하는 기준"

```yaml
- name: product_category
  synonyms: ["제품군", "상품 카테고리"]  # 사용자가 다르게 부를 수 있는 이름들
  description: "상품이 속한 카테고리"     # AI가 이해할 수 있는 설명
  expr: PRODUCT_CATEGORY              # 실제 테이블의 컬럼명
  data_type: varchar                  # 데이터 타입
  sample_values: ["전자제품", "신발"]    # 실제 데이터 예시
```

**실제 활용 예시:**
- 사용자 질문: "전자제품 매출이 얼마야?"
- AI 이해: `product_category = '전자제품'`으로 필터링

### 4.2 Time Dimensions (시간 차원) - "언제"를 나타내는 기준

```yaml
- name: order_date
  synonyms: ["주문일", "구매날짜"]
  description: "고객이 주문한 날짜"
  expr: ORDER_DATE
  data_type: date
```

**실제 활용 예시:**
- 사용자 질문: "지난 달 주문은 몇 건이야?"
- AI 이해: `ORDER_DATE`에서 지난 달 범위로 필터링

### 4.3 Measures (측정값) - "얼마나"를 나타내는 수치

```yaml
- name: total_revenue
  synonyms: ["총매출", "매출액"]
  description: "상품별 총 매출액"
  expr: ITEM_REVENUE
  data_type: number
  default_aggregation: sum  # 기본 집계 방법 (합계)
```

**실제 활용 예시:**
- 사용자 질문: "총 매출이 얼마야?"
- AI 이해: `SUM(ITEM_REVENUE)`로 계산

## 5단계: 실제 사용 시나리오

### 5.1 매출 현황 분석

| 자연어 질문 | 생성되는 SQL 개념 | 결과 |
|------------|------------------|------|
| "이번 달 총 매출은 얼마야?" | `SUM(ITEM_REVENUE) WHERE ORDER_DATE = 현재월` | ₩2,495,000 |
| "전자제품 매출이 얼마야?" | `SUM(ITEM_REVENUE) WHERE PRODUCT_CATEGORY = '전자제품'` | ₩2,200,000 |
| "VIP 고객들 매출은?" | `SUM(ITEM_REVENUE) WHERE CUSTOMER_SEGMENT = 'VIP'` | ₩1,275,000 |

### 5.2 상품 분석

| 자연어 질문 | 시맨틱 모델 활용 | 실제 의미 |
|------------|-----------------|----------|
| "어떤 브랜드가 가장 잘 팔려?" | `brand` 차원 + `quantity_sold` 측정값 | 브랜드별 판매량 순위 |
| "Apple 제품 이익률은?" | `brand` 필터 + `profit/total_revenue` 계산 | Apple 제품의 수익성 |
| "신발 카테고리 평균 단가는?" | `product_category` 필터 + `unit_price` 평균 | 신발의 평균 판매가격 |

### 5.3 시간별 분석

| 자연어 질문 | Time Dimension 활용 | 분석 결과 |
|------------|-------------------|----------|
| "지난주 대비 이번주 매출 증가율은?" | `order_date` 주별 그룹핑 | 주간 성장률 계산 |
| "1월 15일 이후 주문 건수는?" | `order_date >= '2024-01-15'` | 특정 날짜 이후 주문량 |
| "요일별 매출 패턴은?" | `DAYOFWEEK(order_date)` | 요일별 매출 트렌드 |

### 5.4 복합 조건 분석

| 자연어 질문 | 여러 구성요소 조합 |
|------------|------------------|
| "서울 지역 VIP 고객의 전자제품 매출은?" | `region` + `customer_segment` + `product_category` |
| "신용카드로 결제한 주문 중 할인이 적용된 건수는?" | `payment_method` + `discount_amount > 0` |
| "취소되지 않은 주문의 평균 배송비는?" | `order_status` 필터 + `shipping_cost` 평균 |

### 5.5 대화형 분석 (Multi-turn)

```
👤 사용자: "2024년 1월 총 매출은 얼마야?"
🤖 Cortex: "2024년 1월 총 매출은 2,495,000원입니다."

👤 사용자: "그 중에서 전자제품은?"
🤖 Cortex: "전자제품 매출은 2,200,000원으로 전체의 88.2%입니다."

👤 사용자: "Apple과 Samsung 매출 비교해줘"
🤖 Cortex: "Apple: 1,200,000원, Samsung: 1,000,000원입니다."
```

## 6단계: base_table 심화 이해

### 6.1 SALES_SUMMARY 뷰가 필요한 이유

#### ❌ 문제: 물리적 테이블들이 분리되어 있음

```sql
-- 매출 분석을 하려면 4개 테이블을 조인해야 함

-- 주문 정보만 있는 테이블
SELECT * FROM ECOMMERCE_DB.SALES.ORDERS;
/*
ORDER_ID | CUSTOMER_ID | ORDER_DATE | ORDER_AMOUNT | STATUS
ORD001   | C001        | 2024-01-15 | 1200.00      | 배송완료
*/

-- 상품 정보만 있는 테이블  
SELECT * FROM ECOMMERCE_DB.SALES.ORDER_ITEMS;
/*
ORDER_ITEM_ID | ORDER_ID | PRODUCT_ID | QUANTITY | UNIT_PRICE
OI001         | ORD001   | P001       | 1        | 1200.00
*/
```

#### ✅ 해결책: SALES_SUMMARY 뷰로 모든 정보를 통합

```sql
-- 결과: 모든 정보가 하나의 뷰에 통합됨
SELECT * FROM ECOMMERCE_DB.SALES.SALES_SUMMARY;
/*
ORDER_ID | ORDER_DATE | CUSTOMER_NAME | CUSTOMER_SEGMENT | PRODUCT_CATEGORY | BRAND | ITEM_REVENUE | ITEM_PROFIT
ORD001   | 2024-01-15 | 김철수         | VIP             | 전자제품          | Apple | 1200.00      | 400.00
ORD002   | 2024-01-16 | 이영희         | 일반            | 신발             | Nike  | 120.00       | 60.00
*/
```

### 6.2 자연어 → SQL 변환 과정

#### 🗣️ 사용자 질문: "전자제품 매출이 얼마야?"

**시맨틱 모델이 없다면 (복잡한 조인 필요):**
```sql
-- Cortex Analyst가 4개 테이블을 조인해야 함
SELECT SUM(oi.TOTAL_PRICE) as 전자제품_매출
FROM ECOMMERCE_DB.SALES.ORDERS o
JOIN ECOMMERCE_DB.SALES.ORDER_ITEMS oi ON o.ORDER_ID = oi.ORDER_ID
JOIN ECOMMERCE_DB.MASTER.PRODUCTS p ON oi.PRODUCT_ID = p.PRODUCT_ID
JOIN ECOMMERCE_DB.MASTER.CUSTOMERS c ON o.CUSTOMER_ID = c.CUSTOMER_ID
WHERE p.CATEGORY = '전자제품'
  AND o.STATUS != '주문취소';
```

**시맨틱 모델이 있다면 (단순한 쿼리):**
```sql
-- Cortex Analyst가 뷰 하나만 사용하면 됨
SELECT SUM(ITEM_REVENUE) as 전자제품_매출
FROM ECOMMERCE_DB.SALES.SALES_SUMMARY
WHERE PRODUCT_CATEGORY = '전자제품'
  AND ORDER_STATUS != '주문취소';
```

### 6.3 단계별 분석 과정

#### 1단계: 사용자 질문 분석
```
"전자제품 매출이 얼마야?"
↓
키워드 추출:
- "전자제품" → product_category 차원
- "매출" → total_revenue 측정값
- "얼마" → SUM 집계 함수
```

#### 2단계: 시맨틱 모델 매핑
```yaml
# 시맨틱 모델에서 찾는 정보:
dimensions:
  - name: product_category
    synonyms: ["제품군", "상품 카테고리", "카테고리"]
    expr: PRODUCT_CATEGORY  # ← 실제 컬럼명

measures:
  - name: total_revenue
    synonyms: ["총매출", "매출액", "매출"]
    expr: ITEM_REVENUE      # ← 실제 컬럼명
    default_aggregation: sum
```

#### 3단계: base_table 확인
```yaml
base_table:
  database: ECOMMERCE_DB
  schema: SALES
  table: SALES_SUMMARY    # ← 여기서 데이터 가져옴
```

#### 4단계: SQL 생성
```sql
SELECT SUM(ITEM_REVENUE)           -- measures.total_revenue.expr
FROM ECOMMERCE_DB.SALES.SALES_SUMMARY  -- base_table
WHERE PRODUCT_CATEGORY = '전자제품'     -- dimensions.product_category.expr
```

### 6.4 핵심 포인트

#### 🎯 SALES_SUMMARY 뷰의 역할:
1. **데이터 통합**: 4개 분리된 테이블 → 1개 통합 뷰
2. **성능 향상**: 런타임 조인 없이 미리 조인된 데이터 사용  
3. **단순화**: 복잡한 관계 → 단순한 컬럼 참조
4. **일관성**: 항상 같은 조인 조건으로 데이터 조회

#### 🔄 작업 흐름:
```
물리적 테이블들 → 통합 뷰 생성 → 시맨틱 모델 정의 → Cortex Analyst 사용
ORDERS         →                →                  →
ORDER_ITEMS    → SALES_SUMMARY  → semantic.yaml   → 자연어 질문 답변
PRODUCTS       →                →                  →
CUSTOMERS      →                →                  →
```

#### ⚡ 성능 비교:
| 방식 | 조인 테이블 수 | 실행 시간 | 복잡도 |
|------|---------------|-----------|--------|
| 직접 조인 | 4개 | 느림 | 높음 |
| 뷰 사용 | 1개 | 빠름 | 낮음 |

## 7단계: 다양한 base_table 설정 방법

### 7.1 옵션 1: 통합 뷰 사용 (추천) - 가장 간단하고 효율적

```yaml
tables:
  - name: sales_analysis  # 논리적 테이블명
    description: "매출 분석용 통합 데이터"
    base_table:
      database: ECOMMERCE_DB
      schema: SALES
      table: SALES_SUMMARY    # ← 여러 테이블을 조인한 뷰 사용
    
    dimensions:
      - name: product_category
        expr: PRODUCT_CATEGORY  # 뷰의 컬럼명 직접 사용
      - name: customer_segment  
        expr: CUSTOMER_SEGMENT  # 뷰의 컬럼명 직접 사용
    
    measures:
      - name: revenue
        expr: ITEM_REVENUE      # 뷰의 컬럼명 직접 사용
```

### 7.2 옵션 2: 단일 테이블 사용 - 단순한 경우

```yaml
tables:
  - name: order_summary
    description: "주문 기본 정보"
    base_table:
      database: ECOMMERCE_DB
      schema: SALES  
      table: ORDERS           # ← 단일 테이블 사용
    
    dimensions:
      - name: order_status
        expr: STATUS
      - name: payment_method
        expr: PAYMENT_METHOD
    
    measures:
      - name: order_amount
        expr: ORDER_AMOUNT
```

### 7.3 옵션 3: 여러 논리적 테이블 + 관계 정의 - 복잡한 경우

```yaml
tables:
  # 첫 번째 논리적 테이블
  - name: orders
    description: "주문 정보" 
    base_table:
      database: ECOMMERCE_DB
      schema: SALES
      table: ORDERS           # ← 주문 테이블
    
    dimensions:
      - name: order_status
        expr: STATUS
    measures:
      - name: order_amount
        expr: ORDER_AMOUNT
  
  # 두 번째 논리적 테이블  
  - name: products
    description: "상품 정보"
    base_table:
      database: ECOMMERCE_DB
      schema: MASTER
      table: PRODUCTS         # ← 상품 테이블
    
    dimensions:
      - name: product_category
        expr: CATEGORY
      - name: brand
        expr: BRAND

# 테이블 간 관계 정의 (조인 조건)
relationships:
  - name: order_to_product
    left_table: orders
    right_table: products
    relationship_columns:
      - left_column: PRODUCT_ID
        right_column: PRODUCT_ID
    join_type: inner
    relationship_type: many_to_one
```

### 7.4 각 방법의 장단점

| 방법 | 장점 | 단점 |
|------|------|------|
| **옵션 1 (통합 뷰)** | • 가장 간단함<br>• 성능이 좋음<br>• 시맨틱 모델이 단순함 | • 뷰를 미리 만들어야 함<br>• 데이터 중복 가능성 |
| **옵션 2 (단일 테이블)** | • 매우 단순함<br>• 뷰 생성 불필요 | • 제한적인 분석만 가능<br>• 다른 테이블 정보 활용 불가 |
| **옵션 3 (여러 테이블)** | • 가장 유연함<br>• 정규화된 구조 유지<br>• 복잡한 분석 가능 | • 설정이 복잡함<br>• 런타임 조인으로 성능 저하 가능 |

## 8단계: 실제 배포 및 사용

### 8.1 시맨틱 모델 배포

```sql
-- 1. 시맨틱 모델 저장을 위한 스테이지 생성
CREATE DATABASE IF NOT EXISTS SEMANTIC_MODELS;
CREATE SCHEMA IF NOT EXISTS SEMANTIC_MODELS.DEFINITIONS;

-- 스테이지 생성 (YAML 파일 저장용)
CREATE STAGE IF NOT EXISTS SEMANTIC_MODELS.DEFINITIONS.ECOMMERCE_STAGE
DIRECTORY = (ENABLE = TRUE)
COMMENT = '이커머스 분석용 시맨틱 모델 저장소';

-- 2. 권한 설정
GRANT USAGE ON DATABASE SEMANTIC_MODELS TO ROLE DATA_ANALYST;
GRANT USAGE ON SCHEMA SEMANTIC_MODELS.DEFINITIONS TO ROLE DATA_ANALYST;
GRANT READ ON STAGE SEMANTIC_MODELS.DEFINITIONS.ECOMMERCE_STAGE TO ROLE DATA_ANALYST;

-- 기본 테이블에 대한 SELECT 권한
GRANT SELECT ON ALL TABLES IN SCHEMA ECOMMERCE_DB.SALES TO ROLE DATA_ANALYST;
GRANT SELECT ON ALL TABLES IN SCHEMA ECOMMERCE_DB.MASTER TO ROLE DATA_ANALYST;
```

### 8.2 Streamlit 애플리케이션 예제

```python
import streamlit as st
import snowflake.connector
import requests
import pandas as pd

# Snowflake 연결 설정
def init_connection():
    return snowflake.connector.connect(
        user=st.secrets["user"],
        password=st.secrets["password"],
        account=st.secrets["account"],
        warehouse=st.secrets["warehouse"],
        database=st.secrets["database"],
        schema=st.secrets["schema"]
    )

# Cortex Analyst API 호출 함수
def call_cortex_analyst(question, conversation_history=None):
    """
    Cortex Analyst API를 호출하여 자연어 질문에 대한 SQL과 답변을 생성합니다.
    """
    account_url = f"https://{st.secrets['account']}.snowflakecomputing.com"
    api_url = f"{account_url}/api/v2/cortex/analyst/message"
    
    headers = {
        "Authorization": f"Bearer {st.session_state.get('auth_token')}",
        "Content-Type": "application/json"
    }
    
    if conversation_history:
        messages = conversation_history + [{
            "role": "user",
            "content": [{"type": "text", "text": question}]
        }]
    else:
        messages = [{
            "role": "user", 
            "content": [{"type": "text", "text": question}]
        }]
    
    payload = {
        "messages": messages,
        "semantic_model_file": "@SEMANTIC_MODELS.DEFINITIONS.ECOMMERCE_STAGE/ecommerce_sales_analysis.yaml"
    }
    
    try:
        response = requests.post(api_url, headers=headers, json=payload)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        st.error(f"API 호출 오류: {str(e)}")
        return None

# 메인 애플리케이션
def main():
    st.title("🏢 Snowflake Cortex Analyst")
    st.subheader("자연어로 데이터를 분석하세요")
    
    # 사이드바 설정
    with st.sidebar:
        st.header("설정")
        auth_token = st.text_input("인증 토큰", type="password")
        if auth_token:
            st.session_state['auth_token'] = auth_token
        
        st.markdown("### 예시 질문")
        st.markdown("""
        - 지난 달 총 매출은 얼마인가요?
        - 제품군별 매출 현황을 보여주세요
        - 2024년 1분기 성장률은?
        - 가장 수익성이 높은 지역은?
        """)
    
    # 대화 기록 초기화
    if 'conversation_history' not in st.session_state:
        st.session_state['conversation_history'] = []
    
    # 채팅 인터페이스
    st.markdown("### 💬 데이터 분석 채팅")
    
    # 사용자 입력
    user_question = st.chat_input("데이터에 대해 궁금한 것을 물어보세요...")
    
    if user_question and st.session_state.get('auth_token'):
        # 사용자 질문을 채팅에 표시
        with st.chat_message("user"):
            st.write(user_question)
        
        # Cortex Analyst 호출
        with st.chat_message("assistant"):
            with st.spinner("분석 중..."):
                response = call_cortex_analyst(
                    user_question, 
                    st.session_state['conversation_history']
                )
                
                if response and 'message' in response:
                    message = response['message']
                    
                    for content in message['content']:
                        if content['type'] == 'text':
                            st.write(content['text'])
                        elif content['type'] == 'sql':
                            st.subheader("생성된 SQL 쿼리:")
                            st.code(content['statement'], language='sql')

if __name__ == "__main__":
    main()
```

## 📚 추가 자료

### 관련 문서
- [Snowflake Cortex Analyst 공식 문서](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst)
- [시맨틱 모델 사양](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst/semantic-model-spec)
- [Snowflake Samples GitHub](https://github.com/Snowflake-Labs/sf-samples/tree/main/samples/cortex-analyst)

### 핵심 요약

1. **물리적 테이블 생성**: 실제 비즈니스 데이터를 담은 정규화된 테이블들
2. **통합 뷰 생성**: 분석을 위해 여러 테이블을 조인한 `SALES_SUMMARY` 뷰
3. **시맨틱 모델 정의**: 비즈니스 용어와 데이터베이스 컬럼을 매핑하는 YAML 파일
4. **자연어 질문**: 사용자가 SQL 없이도 데이터를 분석할 수 있는 인터페이스

### 성공을 위한 팁

- **작은 범위부터 시작**: 하나의 핵심 비즈니스 영역부터 시맨틱 모델 구축
- **반복적 개선**: 사용자 피드백을 바탕으로 지속적으로 시맨틱 모델 개선
- **명확한 동의어 제공**: 사용자가 다양하게 표현할 수 있는 비즈니스 용어들을 충분히 포함
- **검증된 쿼리 활용**: 자주 묻는 질문들을 미리 정의하여 일관된 답변 제공
