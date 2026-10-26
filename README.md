<p align="center"><a href="https://apify.com/DevilScrapes/2gis-places-scraper"><img src="assets/banner.svg" alt="2GIS Places Scraper — JavaScript example" width="100%"></a></p>

# 2GIS Scraper for Node.js: local business search inside your app or backend

<p align="center">
<a href="https://apify.com/DevilScrapes/2gis-places-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/2gis-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/2gis-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

Use this repository to call the **[2GIS Places Scraper](https://apify.com/DevilScrapes/2gis-places-scraper)** from JavaScript. A single ES module wraps `apify-client`: your service passes a list of 2GIS city slugs plus category keywords, Apify runs the collection in the cloud, and you receive one plain object per outlet: branch id, the chain's organization id, rubrics, address with building and floor hints, WGS84 coordinates, rating, review total and per-day hours. Phones, website and e-mail can be added on request. Rows are written to `results.json` and `results.csv`, or you import the call and keep them in memory.

> **Actor:** [2GIS Places Scraper](https://apify.com/DevilScrapes/2gis-places-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [2gis-scraper-python](https://github.com/devilscrapes/2gis-scraper-python)

## 🔥 What you get

- **Cities times queries, in one run**: two city slugs and three keywords become six separate searches, and every row records which `searchCity` and `query` produced it.
- **Branch and brand kept apart**: `placeId` identifies one physical outlet, while `organizationId` is the id its whole chain shares, so counting stores and counting companies are two different `nunique` calls.
- **Each outlet appears once per run**: overlapping keywords such as `кафе` and `кофейня` find the same venues; the second sighting is dropped and never charged.
- **Stops where 2GIS stops**: a single search tops out at five pages of twelve. When 2GIS starts handing back its first page again, the Actor notices the page number mismatch and ends that search rather than emitting copies.
- **Contacts only when you ask**: with `includeContacts` on, one extra request per outlet fills `phone`, `phones`, `website` and `email` where the business has published them; a failed contact page keeps the row without them.
- **A plain-language run summary**: the status message lists every city and keyword with its row count and the reason it stopped, and a flaky page is retried with growing pauses before it is given up on.

## 🎯 Use cases

### Seed a directory or delivery app for a new city

When your product launches in Almaty or Novosibirsk, a worker can pull every cafe, florist and pharmacy for that slug and upsert them by `placeId`. Coordinates go straight into your geospatial index; the 2GIS `url` gives support staff a quick way to verify an entry.

### Refresh place data from a scheduled function

Run `node index.js` from a GitHub Actions schedule, a Kubernetes CronJob or a cloud function timer. Compare `updatedAt` with what you stored last time and only rewrite records that 2GIS itself has changed.

### Enrich CRM leads behind a webhook

An Express route can accept a city and a trade from your CRM, start a run with `includeContacts` set to true and post the resulting business phones and websites back to the record when the Apify run-finished webhook fires.

### Expose chain footprints through an internal API

Group rows by `organizationId` to list each brand's branches with addresses and Sunday hours. A small endpoint serving that grouping answers sales questions such as which chains already operate in both Moscow and Saint Petersburg.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/2gis-scraper-javascript.git
cd 2gis-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [2GIS Places Scraper](https://apify.com/DevilScrapes/2gis-places-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Stripped of CLI handling, the integration is four statements. `call()` resolves once the run reaches a final state, and `listItems()` reads the dataset it wrote.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/2gis-places-scraper').call({"cities": ["moscow", "spb"], "queries": ["кафе", "аптека"], "maxItemsPerQuery": 10, "maxPages": 2, "maxResults": 10, "includeContacts": false, "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` sends `input.json` as it stands. The sample checks cafes and pharmacies in Moscow and Saint Petersburg, two pages per search at most and ten outlets across the run, a cheap way to see real rows. In production you would raise `maxResults`, add slugs such as `kazan` or `dubai`, and turn on `includeContacts` when phones matter.

```json
{
  "cities": [
    "moscow",
    "spb"
  ],
  "queries": [
    "кафе",
    "аптека"
  ],
  "maxItemsPerQuery": 10,
  "maxPages": 2,
  "maxResults": 10,
  "includeContacts": false,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `cities` | array | no | `["moscow"]` | — |
| `queries` | array | no | `["\u043a\u0430\u0444\u0435"]` | — |
| `maxItemsPerQuery` | integer | no | `60` | — |
| `maxPages` | integer | no | `5` | — |
| `maxResults` | integer | no | `1000` | — |
| `includeContacts` | boolean | no | `false` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |
| `proxyCountryCode` | string | no | — | — |

## 📤 Output

Every item is an outlet object. Nullable fields really do arrive as `null`, so optional chaining (`row.workingHours?.Mon`) is your friend. Contact fields stay empty unless `includeContacts` was on and the business lists them. Timestamps are ISO-8601 strings in UTC.

| Field | Type | Always present |
|---|---|---|
| `placeId` | string | yes |
| `name` | string | yes |
| `organizationId` | string or null | no |
| `category` | string or null | no |
| `categories` | array | no |
| `address` | string or null | no |
| `addressComment` | string or null | no |
| `buildingName` | string or null | no |
| `postcode` | string or null | no |
| `city` | string or null | no |
| `district` | string or null | no |
| `latitude` | number or null | no |
| `longitude` | number or null | no |
| `rating` | number or null | no |
| `reviewCount` | integer or null | no |
| `phone` | string or null | no |
| `phones` | array | no |
| `website` | string or null | no |
| `email` | string or null | no |
| `workingHours` | object or null | no |
| `updatedAt` | string or null | no |
| `isPromoted` | boolean | no |
| `url` | string | yes |
| `query` | string | yes |
| `searchCity` | string | yes |
| `scrapedAt` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result emitted | $0.003 |

Prices as configured on 2026-10-01. The [Actor page](https://apify.com/DevilScrapes/2gis-places-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

The script below needs only Node.js built-ins. It groups saved outlets by chain, keeps brands with more than one branch, records the cities they appear in and their Sunday hours, writes `chains.json` and prints a table.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const rows = JSON.parse(await readFile('results.json', 'utf8'));
const byOrg = new Map();
for (const row of rows) {
    const key = row.organizationId ?? row.placeId;
    const entry = byOrg.get(key) ?? { name: row.name, branches: 0, cities: new Set(), open: [] };
    entry.branches += 1;
    entry.cities.add(row.searchCity);
    if (row.workingHours?.Sun) entry.open.push(`${row.address ?? row.placeId}: ${row.workingHours.Sun}`);
    byOrg.set(key, entry);
}

const chains = [...byOrg.values()]
    .filter((entry) => entry.branches > 1)
    .map((entry) => ({ ...entry, cities: [...entry.cities] }))
    .sort((a, b) => b.branches - a.branches);
await writeFile('chains.json', JSON.stringify(chains, null, 2));
console.table(chains.map(({ name, branches, cities }) => ({ name, branches, cities: cities.join(',') })));
```

## ❓ FAQ

### How should my app handle a keyword that finds nothing?

Treat it as data. 2GIS answers an unmatched search with a not-found page, and the Actor records that search as finished with zero rows; the run as a whole stays successful. Only a run where every search errored ends as failed.

### Is it safe to use `placeId` as a primary key?

Yes, it is the branch id 2GIS uses in its own place URLs and the key the Actor deduplicates on. Use `organizationId` as a foreign key when you want one company record with many outlets.

### Will the TypeScript compiler know the row shape?

`apify-client` types the run and dataset objects for you. Item contents are untyped, so declare an interface from the Output table below and cast `items` to it once.

### How do I keep a test cheap?

Keep `maxResults` small, leave `includeContacts` off (it adds one page load per outlet) and cap spending with `node index.js --max-charge 0.5`. The run cannot bill past that amount.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/2gis-places-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [2GIS Places Scraper on Apify Store](https://apify.com/DevilScrapes/2gis-places-scraper)
- [Python version of this example](https://github.com/devilscrapes/2gis-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Yandex Maps Places Scraper](https://apify.com/DevilScrapes/yandex-maps-places-scraper) — the other big Russian map directory, for cross-checking coverage.
- [Yandex Maps Reviews Scraper](https://apify.com/DevilScrapes/yandex-maps-reviews-scraper) — review text behind the star ratings in the same cities.
- [Zomato Restaurants Scraper](https://apify.com/DevilScrapes/zomato-restaurants-scraper) — restaurant listings for markets 2GIS does not reach.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/2gis-places-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with 2GIS or its parent company.
