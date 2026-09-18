# Magento 2 Automatic Persisted Queries (APQ) — maintained fork (SISL)

An Apollo-compatible **Automatic Persisted Queries** implementation for GraphQL in Magento 2.
Instead of sending the whole GraphQL query over and over, the client sends only its **SHA-256
hash**; the full query goes to the server once and is remembered. The result:

- **smaller GraphQL request payloads** (the frontend sends a hash instead of kilobytes of query),
- **queries over GET** instead of POST → cacheable on a CDN / at the edge,
- less network overhead for headless / PWA frontends (Apollo Client, `apollo-link-persisted-queries`).

This is a **maintained fork** of the abandoned `danslo/magento2-module-automatic-persisted-queries`
(last upstream commit: 2023). The original **declares no `require`** in `composer.json` (it mounts
anywhere, untested against newer releases) and **ships no `etc/acl.xml`**, even though `system.xml`
references the `Danslo_Apq::config_apq` resource — which left the admin configuration section
effectively inaccessible. This fork fixes that and is verified on **2.4.9 / PHP 8.4**.

## What the fork fixes
- **Added `etc/acl.xml`** with the `Danslo_Apq::config_apq` resource (under `Magento_Config::config`) — the *Stores → Configuration → General → Automatic Persisted Queries* section is now visible and saveable. (a real bug in the original)
- **Filled in `composer.json`**: `php: ~8.1.0 || … || ~8.5.0`, `magento/framework: >=103.0.4 <104`, `magento/module-graph-ql: >=100.4.0 <101` (instead of no requirements at all).
- Magento 2.4.9 compatibility confirmed with a real test of the full APQ handshake (see below).

## Compatibility
- Magento **2.4.4 – 2.4.9** (Open Source / Adobe Commerce)
- PHP **8.1 – 8.4**

## Installation

```bash
composer require sisl-source/magento2-automatic-persisted-queries
bin/magento module:enable Danslo_Apq
bin/magento cache:enable apq          # cache type for the persisted queries
bin/magento setup:upgrade
bin/magento setup:di:compile   # production mode
```

## How the handshake works (Apollo APQ)
1. The client sends only `extensions.persistedQuery.sha256Hash` (no `query`). If the server does not know the hash → it responds with a `PersistedQueryNotFound` error.
2. The client retries with the full `query` **and** the hash. The server verifies `sha256(query) == hash`, stores the query in the `apq` cache and returns the data.
3. Subsequent requests carrying only the hash hit the cache and return data without sending the query.

## Configuration
**Stores → Configuration → General → Automatic Persisted Queries** — the HTTP codes returned in
edge cases (matching apollo-server behaviour):

| Field | Default |
|------|-----------|
| HTTP code — query not found (GET) | 400 |
| HTTP code — query not found (POST) | 500 |
| HTTP code — invalid SHA (GET) | 400 |
| HTTP code — invalid SHA (POST) | 500 |

The **`apq`** (Automatic Persisted Queries) cache type appears in *System → Cache Management* —
that is also where you clear the remembered queries.

## License
MIT (same as upstream). Fork maintained by [SISL](https://sisl.pl).

---

### Maintained by SISL

Maintained fork by **[SISL](https://sisl.pl)** — [Magento 2 development and modules](https://sisl.pl/moduly-magento). More self-hosted plugins: [SISL Marketplace](https://sisl.pl/sklep).