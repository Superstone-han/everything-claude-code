---
name: clickhouse-io
description: 고성능 분석 작업 부하를 위한 ClickHouse 데이터베이스 패턴, 쿼리 최적화, 분석 및 데이터 엔지니어링 모범 사례.
---

# ClickHouse 분석 패턴

고성능 분석 및 데이터 엔지니어링을 위한 ClickHouse별 패턴입니다.

## 개요

ClickHouse는 온라인 분석 처리(OLAP)용 열 지향 데이터베이스 관리 시스템(DBMS)입니다. 대형 데이터세트에서 빠른 분석 쿼리에 최적화되어 있습니다.

**핵심 기능:**
- 열 지향 저장소
- 데이터 압축
- 병렬 쿼리 실행
- 분산 쿼리
- 실시간 분석

## 테이블 설계 패턴

### MergeTree 엔진 (가장 일반적)

```sql
CREATE TABLE markets_analytics (
    date Date,
    market_id String,
    market_name String,
    volume UInt64,
    trades UInt32,
    unique_traders UInt32,
    avg_trade_size Float64,
    created_at DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, market_id)
SETTINGS index_granularity = 8192;
```

### ReplacingMergeTree (중복 제거)

중복 데이터가 있을 수 있는 데이터에 사용하세요.

### AggregatingMergeTree (사전 집계)

집계 메트릭을 유지 관리하세요.

## 쿼리 최적화 패턴

### 효율적인 필터링

```sql
-- ✅ 좋음: 인덱스된 열을 먼저 사용
SELECT *
FROM markets_analytics
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
ORDER BY date DESC
LIMIT 100;
```

### 집계

```sql
-- ✅ 좋음: ClickHouse별 집계 함수 사용
SELECT
    toStartOfDay(created_at) AS day,
    market_id,
    sum(volume) AS total_volume,
    count() AS total_trades,
    uniq(trader_id) AS unique_traders
FROM trades
WHERE created_at >= today() - INTERVAL 7 DAY
GROUP BY day, market_id
ORDER BY day DESC, total_volume DESC;
```

### 윈도우 함수

```sql
-- 누계 계산
SELECT
    date,
    market_id,
    volume,
    sum(volume) OVER (
        PARTITION BY market_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_volume
FROM markets_analytics
WHERE date >= today() - INTERVAL 30 DAY
ORDER BY market_id, date;
```

## 데이터 삽입 패턴

### 대량 삽입 (권장)

```typescript
// ✅ 일괄 삽입 (효율적)
async function bulkInsertTrades(trades: Trade[]) {
  const values = trades.map(trade => `(
    '${trade.id}',
    '${trade.market_id}',
    '${trade.user_id}',
    ${trade.amount},
    '${trade.timestamp.toISOString()}'
  )`).join(',')

  await clickhouse.query(`
    INSERT INTO trades (id, market_id, user_id, amount, timestamp)
    VALUES ${values}
  `).toPromise()
}
```

### 스트리밍 삽입

연속 데이터 수집을 위한 스트림을 구현하세요.

## 머티리얼라이즈드 뷰

### 실시간 집계

```sql
-- 시간별 통계용 머티리얼라이즈드 뷰 생성
CREATE MATERIALIZED VIEW market_stats_hourly_mv
TO market_stats_hourly
AS SELECT
    toStartOfHour(timestamp) AS hour,
    market_id,
    sumState(amount) AS total_volume,
    countState() AS total_trades,
    uniqState(user_id) AS unique_users
FROM trades
GROUP BY hour, market_id;
```

## 성능 모니터링

### 쿼리 성능

```sql
-- 느린 쿼리 확인
SELECT
    query_id,
    user,
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    memory_usage
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query_duration_ms > 1000
ORDER BY query_duration_ms DESC
LIMIT 10;
```

## 일반적인 분석 쿼리

### 시계열 분석

```sql
-- 일일 활성 사용자
SELECT
    toDate(timestamp) AS date,
    uniq(user_id) AS daily_active_users
FROM events
WHERE timestamp >= today() - INTERVAL 30 DAY
GROUP BY date
ORDER BY date;
```

### 퍼널 분석

전환율을 분석하세요.

### 코호트 분석

사용자 코호트를 분석하세요.

## 데이터 파이프라인 패턴

### ETL 패턴

추출, 변환, 로드를 구현하세요.

### 변경 데이터 캡처 (CDC)

PostgreSQL 변경사항을 수신하고 ClickHouse에 동기화하세요.

## 모범 사례

### 1. 파티셔닝 전략
- 시간별 파티션 (보통 월 또는 일)
- 너무 많은 파티션 피하기 (성능 영향)
- 파티션 키에 DATE 타입 사용

### 2. 정렬 키
- 가장 자주 필터링되는 열을 먼저
- 카디널리티 고려 (높은 카디널리티 먼저)
- 순서가 압축에 영향

**기억하기**: ClickHouse는 분석 작업 부하에서 뛰어납니다. 쿼리 패턴용 테이블을 설계하고, 삽입을 일괄 처리하며, 실시간 집계를 위해 머티리얼라이즈드 뷰를 활용하세요.
