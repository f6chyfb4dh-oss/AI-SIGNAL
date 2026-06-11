# AI-SIGNAL Research Dashboard

AI-SIGNAL är en fristående browserbaserad trading research-dashboard för svenska/Avanza-liknande aktier. Den är byggd som **en enda nedladdningsbar HTML-fil** (`index.html`) med HTML, CSS, JavaScript, backtestmotor, pappershandelsjournal, diagram, drag-and-drop CSV-import och inbyggda demo-datasets.

> **This tool is for research and paper trading only. It is not financial advice and does not guarantee profit.**

## Viktig ändring: CSV är huvudflödet

Live-data från Yahoo Finance eller liknande endpoints är **inte** en tillförlitlig primär datakälla för en standalone statisk HTML-fil i Chrome. Appen är därför omdesignad så att **CSV-import är den rekommenderade och säkraste arbetsgången** för riktiga backtester.

Appen innehåller fortfarande ett frivilligt Yahoo-liveförsök, men det är endast en bekvämlighet. Om det misslyckas visar appen tydligt:

> “Live data failed. Import CSV to backtest real data.”

Om demo-data används visar appen tydligt:

> “You are using demo data, not real market data.”

## Varför Yahoo Finance kan misslyckas i browsern

Yahoo Finance chart endpoint kan fungera i vissa miljöer men misslycka i andra. Vanliga orsaker:

- **CORS:** En statisk HTML-sida i Chrome får bara läsa svar från andra domäner om servern skickar tillåtande CORS-headers. Om endpointen inte tillåter din origin blockeras svaret av browsern.
- **Endpoint-restriktioner:** Inofficiella eller publika endpoints kan ändra krav, cookies, headers, rate limits eller anti-bot-regler utan förvarning.
- **Static HTML limitations:** En fil som körs via `file://` eller en statisk host har ingen backend som kan hantera cookies, sessionsflöden, hemliga nycklar eller server-side proxy.
- **Browser security:** Chrome blockerar cross-origin-läsning för att skydda användare från att webbsidor läser data från andra tjänster utan tillåtelse.

Avanza används inte som datakälla eller execution-koppling. Appen har ingen broker-login, inga riktiga order, ingen Avanza-exekvering och sparar inga API-nycklar i webbläsaren.

## Så importerar du historisk data

1. Exportera historisk OHLCV-data från valfri laglig och licensmässigt tillåten källa.
2. Spara filen som CSV med följande kolumner:

```csv
Date,Open,High,Low,Close,Volume
2024-01-02,100,102,99,101,1200000
2024-01-03,101,103,100,102,1300000
```

3. Öppna `index.html` i Chrome.
4. Dra CSV-filen till drag-and-drop-zonen eller klicka på **Välj CSV-fil**.
5. Kontrollera success-meddelandet: appen visar radantal, datumintervall och symbol/namn.
6. Klicka på **Kör backtest**.

CSV-parsern kräver kolumnerna `Date`, `Open`, `High`, `Low`, `Close`, `Volume`. Datum bör vara i formatet `YYYY-MM-DD`.

## Inbyggda testdata

Appen har tre inbyggda demo-datasets för UI- och strategitest:

- `Demo: trend up`
- `Demo: sideways/choppy`
- `Demo: downtrend`

Samma typ av testdata finns även som CSV-filer i repot:

- `demo-trend-up.csv`
- `demo-sideways-choppy.csv`
- `demo-downtrend.csv`
- `demo-data.csv` (bakåtkompatibel trend-up sample)

Dessa filer är syntetiska och ska inte tolkas som riktig marknadsdata.

## Strategidiagnostik

När data är laddad visar appen:

- **Raw signal count** – antal enkla EMA-baserade råsignaler.
- **Filtered signal count** – antal råsignaler som klarar de strikta filtren.
- **Final trade count** – antal simulerade trades efter risk-, kapital-, cooldown- och dagsregler.

Om resultatet blir 0 trades visar appen vilka filter som blockerade signalerna, till exempel trendfilter, EMA-alignment, RSI, ATR-volatilitet, volymfilter eller för litet kapital för positionen.

## Strategilogik och riskregler

Strategin är gjord för att minska överhandel och drawdown jämfört med en mikrotrading-Pine-strategi.

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

## Öppna i Chrome

1. Klona eller ladda ner repot.
2. Öppna `index.html` direkt i Chrome, eller kör en enkel lokal statisk server:

```bash
python3 -m http.server 8080
```

3. Gå till `http://localhost:8080/index.html`.
4. Importera CSV för riktiga backtester.

En lokal server är ofta bättre än `file://` eftersom browserbeteende, CDN-laddning och felsökning blir tydligare.

## Deploy som statisk cloud page

Ladda upp `index.html` till valfri statisk host, till exempel:

- GitHub Pages
- Netlify
- Vercel static hosting
- Cloudflare Pages
- Azure Static Web Apps
- AWS S3 + CloudFront

Ingen backend krävs för CSV-flödet. Användaren importerar CSV lokalt i browsern. Om du senare vill ha stabil live-data bör du använda en licensierad dataleverantör och vanligtvis en backend/proxy som följer leverantörens villkor.

## API-nycklar och mäklarkopplingar

- Appen kräver ingen API-nyckel för CSV-flödet.
- Appen sparar inga API-nycklar i browsern.
- Appen ansluter inte till Avanza eller någon annan mäklare.
- Appen kan inte placera riktiga order.
- Pappershandelsjournalen sparas endast lokalt i `localStorage`.

## Viktig riskinformation

Startkapitalet 100 SEK är avsiktligt konfigurerbart men är mycket litet för verklig aktiehandel. Courtage, spread, minsta orderstorlek, skatter och avrundning kan dominera resultatet. Appen är därför en research- och utbildningsmiljö, inte en rekommendation att handla.

**This tool is for research and paper trading only. It is not financial advice and does not guarantee profit.**
