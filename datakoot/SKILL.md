---
name: datakoot
description: Official-source data for agents in one call, with no account and no API key. Use when a task needs CVE / CISA KEV / EPSS vulnerability data, npm/PyPI/crates package health, SEC EDGAR filings and insider trades, World Bank / BLS macro data, ECB FX rates, US weather and geocoding, DNS/RDAP domain intel, the Federal Register, or Base on-chain data. Free over MCP (100 calls/day per caller) or $0.01 per call over x402 (USDC on Base).
---

# Datakoot

Datakoot serves 52 tools from official public sources (NVD, CISA, FIRST, OSV, SEC EDGAR, World Bank, BLS, ECB, NWS, USGS, US Census, RDAP, the Federal Register, Base mainnet). No account, no email, no API key.

## Pick a path

1. **Free (MCP, no key):** add the server for the domain you need, e.g. `https://security.datakoot.com/mcp`. Servers: `security`, `package`, `filings`, `economy`, `market`, `weather`, `domain`, `regulatory`, `base`, all at `https://<server>.datakoot.com/mcp`, plus `https://koot.datakoot.com/mcp` for one-call briefs. 100 calls a day per caller across all nine servers.
2. **Pay per call (x402, no account):** `POST https://x402.datakoot.com/api/<tool>` with the tool's arguments as a JSON body (or MCP at `https://x402.datakoot.com/mcp`). The first reply is HTTP 402 with x402 v2 payment terms: **$0.01 USDC on Base** (`eip155:8453`, exact scheme, payTo `0x3de7b2de531053340cf132d29b837e5a0e924177`). Sign, resend with the payment header, get the data. Charged only when the call succeeds; a failed tool call is not settled.
3. **Pro:** $10/month, 50,000 calls, no daily limit: https://datakoot.com/pricing

## Rules of thumb

- Prefer one fused call over several raw ones: `cve_lookup` already joins NVD + CISA KEV + EPSS; `brief_stack` audits a whole dependency list and ranks fix-first by real-world exploitation.
- Responses are JSON; most include a `provenance` block (upstream sources, time served). Cite it when present.
- If the free tier answers "Daily free limit reached", either wait for 00:00 UTC or pay that same call via x402 as above.
- Machine-readable catalog: https://x402.datakoot.com/openapi.json and https://x402.datakoot.com/discovery

## Tools

| Tool | Server | Arguments (`?` = optional) | What it returns | Example arguments |
|---|---|---|---|---|
| `cve_lookup` | Security Intel | `cve_id` | Look up a CVE by ID and get a compact summary: description, CVSS score & severity, vector, CWE weakness, publish date, references — plus whether it… | `{"cve_id": "CVE-2024-3094"}` |
| `known_exploited` | Security Intel | `cve_id`?, `limit`?, `vendor`?, `ransomware_only`? | Check whether a CVE is on the CISA Known Exploited Vulnerabilities (KEV) catalog — confirmed exploited in the wild — or list the most recently added… | `{"limit": 5}` |
| `epss_score` | Security Intel | `cve_id`?, `cve_ids`? | Get the EPSS exploit-probability score (0-1) and percentile for one or more CVEs — the likelihood each is exploited in the next 30 days. Use it to… | `{"cve_ids": ["CVE-2024-3094", "CVE-2021-44228"]}` |
| `package_vulnerabilities` | Security Intel | `ecosystem`, `name`, `version`? | List known vulnerabilities for a software package (optionally a specific version) via OSV. Ecosystems: npm, pypi, cargo, go, maven, rubygems, nuget,… | `{"ecosystem": "npm", "name": "lodash", "version": "4.17.15"}` |
| `audit_dependencies` | Security Intel | `manifest`?, `dependencies`?, `ecosystem`? | Audit a whole dependency manifest for known vulnerabilities in one call. Paste a package.json (as 'manifest'), or pass a 'dependencies' array of… | `{"manifest": "{\"dependencies\": {\"lodash\": \"4.17.15\", \"express\": \"4.17.1\"}}"}` |
| `package_info` | Package Intel | `ecosystem`, `name` | Get core metadata for a package: latest version, description, license, homepage, source repository, maintainers, and whether it is deprecated. Use… | `{"ecosystem": "pypi", "name": "requests"}` |
| `package_versions` | Package Intel | `ecosystem`, `name`, `limit`? | List recent released versions of a package with release dates (most recent first). Helps an agent pick a version or check how actively it is… | `{"ecosystem": "npm", "name": "react", "limit": 5}` |
| `package_downloads` | Package Intel | `ecosystem`, `name` | Get popularity/download statistics for a package (recent download counts). Useful for judging how widely used and battle-tested a dependency is.… | `{"ecosystem": "npm", "name": "react"}` |
| `package_dependencies` | Package Intel | `ecosystem`, `name`, `version`? | List the direct dependencies of a specific package version (defaults to latest), via deps.dev. Lets an agent understand what a package pulls in… | `{"ecosystem": "cargo", "name": "serde"}` |
| `package_health` | Package Intel | `ecosystem`, `name` | A composite trust/health check for a package an agent is considering: latest-release recency, deprecation status, license present, maintainer count,… | `{"ecosystem": "npm", "name": "left-pad"}` |
| `package_search` | Package Intel | `ecosystem`, `query`, `limit`? | Search for packages by keyword and get the top matches with descriptions. Supported ecosystems: npm and cargo (PyPI has no public search API). | `{"ecosystem": "npm", "query": "mcp server", "limit": 5}` |
| `company_lookup` | Filings Intel | `query` | Resolve a stock ticker or company name to its SEC identity: CIK number, official name, ticker(s), exchange(s), industry (SIC), and fiscal year end.… | `{"query": "NVDA"}` |
| `recent_filings` | Filings Intel | `query`, `form_type`?, `limit`? | List a company's most recent SEC filings (10-K annual reports, 10-Q quarterly, 8-K material events, S-1 IPO registrations, etc.) with filing dates… | `{"query": "TSLA", "limit": 5}` |
| `filing_search` | Filings Intel | `query`, `form_type`?, `limit`? | Full-text search across all EDGAR filings for a keyword or phrase (e.g. a risk factor, product name, or executive). Returns matching filings with… | `{"query": "artificial intelligence", "form_type": "8-K", "limit": 5}` |
| `financials` | Filings Intel | `query`, `concept`? | Get reported financial facts for a company from XBRL data in its filings: revenue, net income, total assets, liabilities, equity, EPS, and cash —… | `{"query": "AAPL", "concept": "Revenues"}` |
| `insider_transactions` | Filings Intel | `query`, `limit`? | List recent insider ownership filings (Form 3/4/5) for a company — the filings officers, directors, and 10% owners submit when they buy or sell… | `{"query": "MSFT", "limit": 5}` |
| `country_indicator` | Economy Intel | `country`, `indicator`, `years`? | Get a macroeconomic indicator for a country over recent years (World Bank). Indicators: gdp, gdp_per_capita, gdp_growth, inflation, unemployment,… | `{"country": "DE", "indicator": "inflation", "years": 5}` |
| `country_profile` | Economy Intel | `country` | Get a snapshot of a country's key macro indicators (latest available values): GDP, GDP per capita, GDP growth, inflation, unemployment, and… | `{"country": "JP"}` |
| `compare_countries` | Economy Intel | `countries`, `indicator` | Compare one indicator across several countries (latest available value each). Good for ranking or benchmarking economies. | `{"countries": ["US", "CN", "IN"], "indicator": "gdp"}` |
| `us_series` | Economy Intel | `series` | Get a key US economic time series. From the Bureau of Labor Statistics: us_unemployment_rate, us_cpi, us_nonfarm_payrolls, us_labor_participation,… | `{"series": "unemployment"}` |
| `list_indicators` | Economy Intel | none | List the indicators this server supports (World Bank + US BLS), with descriptions. Call this to discover what you can query. | `{}` |
| `fx_rates` | Market Intel | `base`?, `symbols`? | Get the latest foreign-exchange rates for a base currency (default USD), optionally limited to specific target currencies. Source: European Central… | `{"base": "USD", "symbols": "EUR,GBP,JPY"}` |
| `fx_convert` | Market Intel | `amount`, `from`, `to` | Convert an amount from one currency to another at the latest ECB reference rate. Returns the rate, the converted result, and the rate date. | `{"amount": 250, "from": "GBP", "to": "USD"}` |
| `fx_historical` | Market Intel | `date`, `base`?, `symbols`? | Get foreign-exchange rates for a base currency on a specific past date (YYYY-MM-DD). ECB publishes rates on business days; a weekend/holiday date… | `{"date": "2026-01-15", "base": "EUR", "symbols": "USD"}` |
| `fx_timeseries` | Market Intel | `start`, `end`, `base`?, `symbols` | Get a time series of exchange rates for a base currency over a date range (max ~1 year), useful for trend analysis. Provide start and end as… | `{"start": "2026-09-01", "end": "2026-09-10", "base": "USD", "symbols": "EUR"}` |
| `fx_currencies` | Market Intel | none | List the currencies supported for exchange-rate lookups, with their full names. | `{}` |
| `geocode` | Weather & Geo | `address` | Convert a US street address OR a city/town name into latitude/longitude coordinates, using US Census data. Accepts both '1600 Pennsylvania Ave NW,… | `{"address": "1600 Pennsylvania Ave NW, Washington, DC"}` |
| `weather_forecast` | Weather & Geo | `lat`, `lon`, `hourly`? | Get a multi-day weather forecast for a US location by latitude/longitude (National Weather Service). Returns named periods (Today, Tonight, ...) with… | `{"lat": 40.27, "lon": -76.88}` |
| `weather_current` | Weather & Geo | `lat`, `lon` | Get the latest observed weather conditions for a US location by latitude/longitude, from the nearest NWS observation station: temperature, humidity,… | `{"lat": 40.27, "lon": -76.88}` |
| `weather_alerts` | Weather & Geo | `area` | List active NWS weather alerts (warnings, watches, advisories) for a US state or marine area. Pass a two-letter state code like CA, TX, FL. Returns… | `{"area": "PA"}` |
| `earthquakes` | Weather & Geo | `min_magnitude`?, `days`?, `lat`?, `lon`?, `radius_km`?, `limit`? | List recent earthquakes worldwide from the USGS feed. Filter by minimum magnitude, look-back window in days, and optionally a bounding box or a… | `{"min_magnitude": 4.5, "days": 3, "limit": 5}` |
| `elevation` | Weather & Geo | `lat`, `lon` | Get the ground elevation at a latitude/longitude within the US, from the USGS Elevation Point Query Service. Returns elevation in feet and meters. | `{"lat": 39.7392, "lon": -104.9903}` |
| `domain_report` | Domain & Company | `domain` | One-call full dossier on a domain: registration (RDAP), DNS records, email deliverability, and website tech stack, combined into a single report. Use… | `{"domain": "anthropic.com"}` |
| `domain_intel` | Domain & Company | `domain` | Registration intelligence for a domain via RDAP (the modern WHOIS). Returns registrar, creation/expiration/last-changed dates, domain age in years,… | `{"domain": "github.com"}` |
| `dns_lookup` | Domain & Company | `domain`, `types`? | Look up DNS records for a domain via Cloudflare DNS-over-HTTPS. Returns A, AAAA, MX, NS, TXT, CNAME, and SOA records. Use to see where a domain is… | `{"domain": "cloudflare.com", "types": ["A", "MX"]}` |
| `email_deliverability` | Domain & Company | `domain_or_email` | Assess whether a domain (or the domain of an email address) can receive mail and how strong its sender authentication is. Returns MX presence, SPF… | `{"domain_or_email": "google.com"}` |
| `tech_stack` | Domain & Company | `domain` | Fingerprint the technology behind a website by fetching its homepage. Detects web server, CMS/framework (WordPress, Shopify, Next.js, etc.), CDN,… | `{"domain": "shopify.com"}` |
| `subdomains` | Domain & Company | `domain` | Discover a domain's public subdomains (app., api., staging., and so on) from Certificate Transparency logs, querying crt.sh and Cert Spotter in… | `{"domain": "datakoot.com"}` |
| `search_documents` | Regulatory Intel | `term`?, `type`?, `agency`?, `since`?, `until`?, `per_page`? | Search the US Federal Register for regulations and notices. Filter by keyword, document type (rule, proposed_rule, notice, presidential_document),… | `{"term": "artificial intelligence", "per_page": 5}` |
| `document` | Regulatory Intel | `document_number` | Get full detail on a single Federal Register document by its document number (e.g. '2026-12811'): abstract, action, effective date, comment deadline,… | `{"document_number": "2024-24283"}` |
| `recent_documents` | Regulatory Intel | `type`?, `agency`?, `days`?, `per_page`? | List the most recent Federal Register documents, optionally filtered by type and issuing agency, over a look-back window. Use this to monitor newly… | `{"days": 3, "per_page": 5}` |
| `executive_orders` | Regulatory Intel | `term`?, `since`?, `per_page`? | Search recent US presidential Executive Orders published in the Federal Register. Optionally filter by keyword and date. Returns EO number,… | `{"per_page": 5}` |
| `agencies` | Regulatory Intel | `query`? | Look up US federal agencies and their slugs (used to filter the other tools by issuing agency). Optionally pass a query to match by name, e.g.… | `{"query": "energy"}` |
| `address_report` | Base Intel | `address` | One-call snapshot of any Base address: ETH balance, USDC balance, transaction count, and whether it is a smart contract or a regular wallet. Use it… | `{"address": "0x3de7b2de531053340cf132d29b837e5a0e924177"}` |
| `token_info` | Base Intel | `token` | ERC-20 token metadata on Base, read on-chain: name, symbol, decimals, and total supply, plus whether the address really behaves like an ERC-20. Use… | `{"token": "0x833589fcd6edb6e08f4c7c32d4f71b54bda02913"}` |
| `token_balance` | Base Intel | `token`, `address` | How much of an ERC-20 token a wallet holds on Base, returned both human-formatted (using the token's decimals) and raw. Pass the token contract… | `{"token": "0x833589fcd6edb6e08f4c7c32d4f71b54bda02913", "address": "0x21217f943Fe780A3E95eBe3dfC95bEF2203cF3bB"}` |
| `gas_now` | Base Intel | none | Current gas price on the Base network (Coinbase's Ethereum L2), in gwei and wei, read live from a Base RPC node. Use before sending a Base… | `{}` |
| `tx_status` | Base Intel | `hash` | Look up a Base transaction by hash and report whether it is pending, succeeded, or failed, with from, to, value, block number, and gas used. Use it… | `{"hash": "0xce51d48ff5b3a2b6c85aa7e6f3ac92c1defe89c9fc6532eaeae4d414d175b068"}` |
| `brief_stack` | Koot briefs | `packages`?, `ecosystem`?, `manifest`? | One call answers 'is my project safe?'. Checks every package for known vulnerabilities, tells you which are being EXPLOITED IN THE WILD right now… | `{"packages": ["lodash@4.17.15", "express@4.17.1"]}` |
| `brief_company` | Koot briefs | `company` | One call briefs you on a US public company: SEC profile, recent filings (8-K material events flagged), insider trades and reported financials. Give a… | `{"company": "TSLA"}` |
| `brief_domain` | Koot briefs | `domain` | One call checks a domain: registration and DNS, email security (SPF/DKIM/DMARC), tech stack, and subdomains from certificate logs. | `{"domain": "stripe.com"}` |
| `brief_place` | Koot briefs | `place` | One call briefs you on a US place: current conditions, forecast, active weather alerts for the state, nearby earthquakes and elevation. | `{"place": "Harrisburg, PA"}` |

## Example (pay per call)

```bash
# 1) ask; you get HTTP 402 with the payment terms in the PAYMENT-REQUIRED header
curl -i -X POST https://x402.datakoot.com/api/cve_lookup -H 'content-type: application/json' -d '{"cve_id":"CVE-2024-3094"}'
# 2) sign an EIP-3009 USDC transfer for the quoted amount with your x402 client and resend with the PAYMENT-SIGNATURE header
```

Any x402 v2 client works (for example the official `x402-fetch` / `x402-axios` packages, or Coinbase AgentKit).
