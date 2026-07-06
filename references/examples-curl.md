# Curl Examples

## Trader discovery with a minimum trade count

```bash
curl -s 'https://partners.centaur.io/api/v1/traders?sourcePlatforms=X&minTrades=5&startTime=2026-03-01T00:00:00.000Z&endTime=2026-03-31T23:59:59.999Z&limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Recent events

```bash
curl -s 'https://partners.centaur.io/api/v1/events?limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Messages in a time window

```bash
curl -s 'https://partners.centaur.io/api/v1/messages?sourcePlatforms=X&startTime=2026-03-01T00:00:00.000Z&endTime=2026-03-07T23:59:59.999Z&limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Hydrate source messages by ID

```bash
curl -s 'https://partners.centaur.io/api/v1/messages?ids=<source-message-id>&limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

Use a Source Message ID returned by a message row `id` or event `messageId`. Do not construct one from platform source/message components.

## Generated Channel Narrative Summaries for a UTC day

```bash
curl -s 'https://partners.centaur.io/api/v1/channel-summaries?startTime=2026-05-09T00:00:00.000Z&endTime=2026-05-10T00:00:00.000Z&limit=50' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Generated Aggregate Narrative Summaries for a UTC day

```bash
curl -s 'https://partners.centaur.io/api/v1/aggregate-summaries?startTime=2026-05-09T00:00:00.000Z&endTime=2026-05-10T00:00:00.000Z&limit=50' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Position history

```bash
curl -s 'https://partners.centaur.io/api/v1/positions?positionIds=9001,9002&limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Most active traders of the last 7 days

```bash
curl -s 'https://partners.centaur.io/api/v1/traders/rankings?metric=event_count&limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Best 7D win rate over positions opened in a bounded window

```bash
curl -s 'https://partners.centaur.io/api/v1/traders/rankings?metric=win_rate&timeBasedPerformanceWindow=7D&startTime=2026-06-01T00:00:00.000Z&endTime=2026-07-01T00:00:00.000Z&limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Daily message and event volume per trader

```bash
curl -s 'https://partners.centaur.io/api/v1/activity-summaries?groupBy=trader&interval=day&limit=10' \
  -H "x-api-key: $CENTAUR_API_KEY"
```

## Forward pagination

```bash
curl -s 'https://partners.centaur.io/api/v1/events?limit=25&cursor=<nextCursor>' \
  -H "x-api-key: $CENTAUR_API_KEY"
```
