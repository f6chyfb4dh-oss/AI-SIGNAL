# AI-SIGNAL Research Dashboard

AI-SIGNAL är en fristående browserbaserad trading research-dashboard för svenska/Avanza-liknande aktier. Den är byggd som **en enda nedladdningsbar HTML-fil** (`index.html`) med HTML, CSS, JavaScript, backtestmotor, pappershandelsjournal, diagram och demo-data.

> **This tool is for research and paper trading only. It is not financial advice and does not guarantee profit.**

## Vad appen gör

- Kör direkt i Chrome som en lokal HTML-fil eller som statisk cloud-sida.
- Backtestar en lågfrevent, long-only aktiestrategi med max 5 signaler per dag.
- Simulerar endast pappershandel; appen lägger aldrig riktiga order.
- Ansluter inte till Avanza trading execution eller konto-API.
- Spårar ROI, vinstfrekvens, genomsnittlig vinst/förlust, max drawdown, antal trades och aktiekurva.
- Visar prisdiagram med EMA 5/9/20, köp-/säljmarkörer, aktiekurva, drawdown och daglig trade count.
- Har CSV-import och inbyggd demo-data så att appen fungerar även när externa datakällor misslyckas.
- Innehåller `demo-data.csv` som testdata för CSV-import och samma typ av demo-serie finns inbäddad i HTML-filen.

## Öppna i Chrome

1. Klona eller ladda ner repot.
2. Öppna `index.html` direkt i Chrome:
   - Dubbelklicka på filen, eller
   - Kör en enkel lokal statisk server:

```bash
python3 -m http.server 8080
```

3. Gå till `http://localhost:8080/index.html`.

Appen kan köras direkt från filsystemet, men en lokal server är ofta bättre för browser-API:er och tydligare felsökning.

## Deploy som statisk cloud page

Ladda upp `index.html` till valfri statisk host, till exempel:

- GitHub Pages
- Netlify
- Vercel static hosting
- Cloudflare Pages
- Azure Static Web Apps
- AWS S3 + CloudFront

Ingen backend krävs. Observera att externa dataförfrågningar från en statisk sida kan påverkas av CORS, rate limits eller tredjepartsändringar.

## Data fetching

### Vald primär datakälla

Appen använder Yahoo Finance chart endpoint som primär källa:

```text
https://query1.finance.yahoo.com/v8/finance/chart/{SYMBOL}?interval=1d
```

Exempel på svenska Yahoo-symboler i appen:

- `ERIC-B.ST`
- `VOLV-B.ST`
- `INVE-B.ST`
- `SEB-A.ST`

### Varför Yahoo Finance?

- Praktiskt för svenska aktiesymboler med `.ST`-suffix.
- Kräver ingen API-nyckel i den här implementationen.
- Returnerar daglig OHLCV-data i JSON-format.
- Är enklare för en fristående HTML-app än mäklar-/kontoanslutningar.

### Avanza-data

Avanza används inte som primär datakälla i appen. Skälen är:

- Appen ska inte ansluta till mäklarexekvering eller konto.
- En statisk HTML-sida bör inte hantera mäklarinloggning, sessionscookies eller orderbehörighet.
- Tillförlitlig publik historikdata från Avanza med stabil CORS för direkt browseranrop är inte ett säkert antagande.

### API-nyckel

- Yahoo-läget i `index.html` kräver ingen API-nyckel.
- Om du senare vill använda Alpha Vantage, Twelve Data, Polygon eller annan leverantör krävs normalt API-nyckel och eventuellt proxy/backend för att skydda nyckeln.

### CORS och begränsningar

- Direkt browserfetch kan sluta fungera om dataleverantören ändrar CORS, cookies, rate limits eller endpoint.
- Yahoo Finance chart endpoint är praktisk men inte en garanterad officiell SLA-produkt för din applikation.
- För produktion bör du överväga en licensierad dataleverantör eller en egen backend/proxy som följer leverantörens villkor.

### Fallback om API misslyckas

Appen faller automatiskt tillbaka till inbyggd demo-data om livehämtning misslyckas. Du kan också importera CSV.

## CSV-import

Använd CSV-filen om live data misslyckas eller om du har egen licensierad historikdata.

Krav på kolumner:

```csv
Date,Open,High,Low,Close,Volume
2024-01-02,100,102,99,101,1200000
2024-01-03,101,103,100,102,1300000
```

Kolumnnamnen ska finnas i första raden. Appen tolkar daglig OHLCV-data och kör samma backtestmotor som för livehämtad data.

## Strategilogik

Den nya logiken är gjord för att minska överhandel och drawdown jämfört med en mikrotrading-Pine-strategi.

Filter och riskregler:

- EMA 5, EMA 9, EMA 20, EMA 50 och EMA 100.
- ATR 14 för stop loss, take profit, trailing stop och volatilitetsfilter.
- RSI 14 för momentum-/överhettningsfilter.
- Högre tidsram approximeras i daglig data med lång trend via EMA 50/100.
- EMA-trend alignment kräver `EMA5 > EMA9 > EMA20` och pris över lång trend.
- Volymfilter undviker lågvolym-/flatperioder.
- Minsta reward/risk måste vara uppfyllt.
- Max 5 trades per dag.
- Minsta cooldown mellan trades.
- Stoppar nya entries när dagens förlustgräns är nådd.
- En öppen position åt gången.
- Long-only som standard.
- Ingen hävstång.
- Ingen blankning som standard.
- Ingen martingale.
- Ingen win-streak-riskökning.

Position sizing använder fast fraktionell risk per trade och begränsas av tillgängligt kapital.

## Viktig riskinformation

Startkapitalet 100 SEK är avsiktligt konfigurerbart men är mycket litet för verklig aktiehandel. Courtage, spread, minsta orderstorlek, skatter och avrundning kan dominera resultatet. Appen är därför en research- och utbildningsmiljö, inte en rekommendation att handla.

**This tool is for research and paper trading only. It is not financial advice and does not guarantee profit.**
