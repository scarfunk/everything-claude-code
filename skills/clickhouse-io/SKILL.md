---
name: clickhouse-io
description: 고성능 분석 워크로드를 위한 ClickHouse 데이터베이스 패턴, 쿼리 최적화, 분석, 데이터 엔지니어링 모범 사례.
---

# ClickHouse 분석 패턴

고성능 분석 및 데이터 엔지니어링을 위한 ClickHouse 전용 패턴.

## 개요

ClickHouse는 온라인 분석 처리(OLAP)를 위한 컬럼 지향 데이터베이스 관리 시스템(DBMS)입니다. 대규모 데이터셋에 대한 빠른 분석 쿼리에 최적화되어 있습니다.

**주요 기능:**
- 컬럼 지향 저장
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

```sql
-- 중복이 있을 수 있는 데이터용 (예: 여러 소스에서)
CREATE TABLE user_events (
    event_id String,
    user_id String,
    event_type String,
    timestamp DateTime,
    properties String
) ENGINE = ReplacingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, event_id, timestamp)
PRIMARY KEY (user_id, event_id);
```

### AggregatingMergeTree (사전 집계)

```sql
-- 집계된 메트릭 유지용
CREATE TABLE market_stats_hourly (
    hour DateTime,
    market_id String,
    total_volume AggregateFunction(sum, UInt64),
    total_trades AggregateFunction(count, UInt32),
    unique_users AggregateFunction(uniq, String)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, market_id);

-- 집계된 데이터 쿼리
SELECT
    hour,
    market_id,
    sumMerge(total_volume) AS volume,
    countMerge(total_trades) AS trades,
    uniqMerge(unique_users) AS users
FROM market_stats_hourly
WHERE hour >= toStartOfHour(now() - INTERVAL 24 HOUR)
GROUP BY hour, market_id
ORDER BY hour DESC;
```

## 쿼리 최적화 패턴

### 효율적인 필터링

```sql
-- ✅ 좋음: 인덱싱된 컬럼을 먼저 사용
SELECT *
FROM markets_analytics
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
ORDER BY date DESC
LIMIT 100;

-- ❌ 나쁨: 인덱싱되지 않은 컬럼으로 먼저 필터
SELECT *
FROM markets_analytics
WHERE volume > 1000
  AND market_name LIKE '%election%'
  AND date >= '2025-01-01';
```

### 집계

```sql
-- ✅ 좋음: ClickHouse 전용 집계 함수 사용
SELECT
    toStartOfDay(created_at) AS day,
    market_id,
    sum(volume) AS total_volume,
    count() AS total_trades,
    uniq(trader_id) AS unique_traders,
    avg(trade_size) AS avg_size
FROM trades
WHERE created_at >= today() - INTERVAL 7 DAY
GROUP BY day, market_id
ORDER BY day DESC, total_volume DESC;

-- ✅ 백분위수에 quantile 사용 (percentile보다 효율적)
SELECT
    quantile(0.50)(trade_size) AS median,
    quantile(0.95)(trade_size) AS p95,
    quantile(0.99)(trade_size) AS p99
FROM trades
WHERE created_at >= now() - INTERVAL 1 HOUR;
```

## 데이터 삽입 패턴

### 대량 삽입 (권장)

```typescript
import { ClickHouse } from 'clickhouse'

const clickhouse = new ClickHouse({
  url: process.env.CLICKHOUSE_URL,
  port: 8123,
  basicAuth: {
    username: process.env.CLICKHOUSE_USER,
    password: process.env.CLICKHOUSE_PASSWORD
  }
})

// ✅ 배치 삽입 (효율적)
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

// ❌ 개별 삽입 (느림)
async function insertTrade(trade: Trade) {
  // 루프에서 이렇게 하지 마세요!
  await clickhouse.query(`
    INSERT INTO trades VALUES ('${trade.id}', ...)
  `).toPromise()
}
```

## 구체화된 뷰

### 실시간 집계

```sql
-- 시간별 통계를 위한 구체화된 뷰 생성
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
  AND event_time >= now() - INTERVAL 1 HOUR
ORDER BY query_duration_ms DESC
LIMIT 10;
```

## 모범 사례

### 1. 파티셔닝 전략
- 시간으로 파티션 (보통 월 또는 일)
- 너무 많은 파티션 피하기 (성능 영향)
- 파티션 키에 DATE 타입 사용

### 2. 정렬 키
- 가장 자주 필터되는 컬럼을 먼저
- 카디널리티 고려 (높은 카디널리티 먼저)
- 순서가 압축에 영향

### 3. 데이터 타입
- 가장 작은 적절한 타입 사용 (UInt32 vs UInt64)
- 반복되는 문자열에 LowCardinality 사용
- 범주형 데이터에 Enum 사용

### 4. 피해야 할 것
- SELECT * (컬럼 명시)
- FINAL (쿼리 전 데이터 병합 대신)
- 너무 많은 JOIN (분석용 비정규화)
- 작은 빈번한 삽입 (대신 배치)

### 5. 모니터링
- 쿼리 성능 추적
- 디스크 사용량 모니터링
- 병합 작업 확인
- 느린 쿼리 로그 검토

**기억하세요**: ClickHouse는 분석 워크로드에 탁월합니다. 쿼리 패턴에 맞게 테이블을 설계하고, 삽입을 배치하고, 실시간 집계를 위해 구체화된 뷰를 활용하세요.
