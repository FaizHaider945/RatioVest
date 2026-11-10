# RatioVest

**Invest by the numbers, not by tips.**

RatioVest is a free, open-source tool for checking whether a company is financially healthy before you buy its shares. You enter the figures from a company's financial statements, and it works out 26 ratios, gives a health score out of 100, checks whether the share price looks fair, and explains every result in plain English.


Built by M. Faiz Haider, BS FinTech student at Air University Islamabad, for the Zindigi Prize 2026.

## What it does

- **26 ratios** in five groups: liquidity, leverage, efficiency, profitability and valuation.
- **A health score out of 100**, plus a separate price check: cheap, fair or expensive.
- **Compare up to four companies** side by side, with the best value in each row highlighted.
- **Common-size analysis**: every item as a percentage of sales or of total assets.
- **Index analysis**: last year set to 100, so you can see what grew and what shrank.
- **A plain-English explanation** under every ratio, and a Learn page with the formulas.
- **A shared database**: once someone enters a company's figures, everyone else can load them by ticker.
- **A PDF reader (beta)** that tries to pull the figures out of an annual report inside your own browser.

## How to use it

1. Open the Analyzer.
2. Either type the numbers from the company's annual report, or search for a ticker someone has already added, or upload an annual report PDF.
3. Check the quick-check panel under the form. It tells you whether the figures fit together.
4. Read the score, the summary and the ratio table.

All amounts are in PKR millions, except the share price and dividend per share.

## How it is built

One self-contained HTML file. No build step, no framework, no server of our own.

| Part | What it uses |
|---|---|
| Page and ratio engine | Plain HTML, CSS and JavaScript in `index.html` |
| 3D scene | [three.js](https://threejs.org) from a CDN |
| PDF reading | [pdf.js](https://mozilla.github.io/pdf.js/) from a CDN, running in the browser |
| Shared database | [Supabase](https://supabase.com) free plan, read through its REST API |
| Hosting | [Netlify](https://www.netlify.com) free plan |

Nothing is uploaded when you open a PDF. It is read inside your browser, and only the extracted numbers are saved, and only if you press "Save for everyone".

## Running your own copy

1. Download `index.html`.
2. Open it in a browser. Everything except the shared database works straight away.
3. For your own database, create a free Supabase project and run this SQL:

```sql
create table companies (ticker text primary key, name text not null, sector text);
create table financials (
  id bigserial primary key,
  ticker text not null references companies(ticker),
  fy int not null,
  status text not null default 'community',
  data jsonb not null,
  source text,
  created_at timestamptz default now(),
  unique (ticker, fy)
);
alter table companies enable row level security;
alter table financials enable row level security;
create policy "read companies" on companies for select using (true);
create policy "add companies" on companies for insert with check (true);
create policy "read financials" on financials for select using (status <> 'rejected');
create policy "add financials" on financials for insert with check (status = 'community');
```

4. In `index.html`, replace the `SB` and `SK` values near the top of the second `<script>` block with your own project URL and publishable (anon) key. Those two values are meant to be public. Never put the `service_role` or secret key in the page.
5. Deploy the file anywhere that serves static files, such as Netlify or GitHub Pages.

## Data quality

Entries saved by visitors are marked **community (unverified)**. They are only as good as the person who entered them. An entry is marked **verified** once a maintainer has checked it against the published annual report.

The PDF reader is **beta**. Pakistani annual reports are 100 to 150 pages and each one is laid out differently, so it often matches the wrong line, for example a percentage from a horizontal-analysis table instead of a rupee figure from the statements. Always check every number it suggests against the report.

## Known limits

- Benchmarks (what counts as strong, okay or weak) are general rules of thumb and are not tuned to any one sector. Compare companies within the same sector.
- Not suitable for banks, insurers or other financial companies. Their balance sheets work differently and ratios like the current ratio mean little for them.
- Share prices are not fetched automatically. Take the price from the PSX website and type it in.
- Turnover ratios use average balances when you provide last year's figures, and closing balances when you do not.

## Contributing

Pull requests are welcome, especially for:

- making the PDF reader find the statement pages before extracting,
- adding sector benchmarks from published annual reports,
- Urdu translations of the ratio explanations,
- multi-year (five-year) index analysis.

Please do not add data copied in bulk from a commercial data provider. Figures entered from a company's own published annual report are fine.

## Not financial advice

RatioVest is an educational tool. Its scores and labels are based only on the numbers entered and on general rules of thumb. They are not a recommendation to buy or sell any security. Do your own research, or speak to a licensed adviser.

## Licence

MIT. See `LICENSE`.
