# Curl Examples

## Recent events

```bash
curl -s 'https://partners.centaur.io/api/v1/events?limit=10' \
  -H "x-api-key: $CENTAUR_PARTNER_API_KEY"
```

## Filter by asset and direction

```bash
curl -s 'https://partners.centaur.io/api/v1/events?assetIds=34&directions=long&limit=10' \
  -H "x-api-key: $CENTAUR_PARTNER_API_KEY"
```

## Bounded time window

```bash
curl -s 'https://partners.centaur.io/api/v1/events?from=2026-02-01T00:00:00.000Z&to=2026-02-07T23:59:59.999Z&limit=25&sortOrder=asc' \
  -H "x-api-key: $CENTAUR_PARTNER_API_KEY"
```

## Forward pagination

```bash
curl -s 'https://partners.centaur.io/api/v1/events?limit=25&after=<cursor>' \
  -H "x-api-key: $CENTAUR_PARTNER_API_KEY"
```
