# Spread Analyzer

Sito statico (un solo file, `index.html`) che confronta due coppie spot Binance tramite il loro rapporto, in entrambe le direzioni (A/B e B/A), e produce un prospetto con fasce di ingresso, take profit, stop loss e probabilità storica di successo.

## Uso

1. Inserisci Coppia A e Coppia B (formato Binance, es. `ETHUSDT`, `BTCUSDT`).
2. Scegli timeframe, numero di candele, finestra della media, durata massima del trade, stop, commissione e slippage per lato.
3. Premi **Analizza**.

Il prospetto contiene, per ogni rapporto e per BUY/SELL: verdetto, stato attuale, livelli di ingresso / take profit / stop loss, probabilità OOS e IS con intervallo di Wilson 95%, e la tabella completa delle soglie testate.

## Metodo

- Dati: candele pubbliche chiuse da `api.binance.com` (fallback `data-api.binance.vision`). Nessuna chiave API.
- Allineamento: solo timestamp presenti in entrambe le coppie; le candele mancanti sono escluse e conteggiate, mai interpolate.
- Segnale: z-score del rapporto su media/deviazione mobili; ingresso al primo attraversamento di |z| ≥ soglia (1; 1,5; 2; 2,5; 3).
- Esecuzione: apertura della barra successiva. TP = ritorno alla media del momento del segnale. SL = soglia + "stop oltre soglia" in z. Uscita a tempo dopo N barre. Uscite valutate sulle chiusure.
- P&L: gamba long meno gamba short a nozionale uguale, meno 4 × (commissione + slippage).
- Validazione: soglia scelta sul primo 70% (IS), risultati riportati anche sul restante 30% (OOS). Meno di 10 trade = esito "insufficiente".

## Limiti

- Il test usa solo chiusure e non modella funding, borrow, spread bid/ask reali né liquidità.
- Un rapporto non stazionario può non tornare alla media: la probabilità storica non è una previsione.
- Non è consulenza finanziaria.

## Provare in locale

```bash
python -m http.server 8000
# apri http://localhost:8000
```

## Pubblicare su GitHub Pages

1. Crea un repository su GitHub e carica `index.html` e `README.md` nella radice.
2. Vai in **Settings → Pages**.
3. In **Build and deployment**, scegli **Deploy from a branch**, branch `main`, cartella `/ (root)`, poi **Save**.
4. Dopo circa un minuto il sito è su `https://<utente>.github.io/<repository>/`.

Se un browser o una rete blocca le richieste a Binance (alcuni paesi o VPN), compare un errore esplicito: il sito non mostra mai dati sostitutivi.
