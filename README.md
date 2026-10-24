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

## Novità: funding spot + perpetual e peggior trade

- **Funding:** il riquadro "Funding spot + perpetual" scarica fino a 1000 rilevazioni da `fapi.binance.com`, mostra il funding mese per mese, il rendimento medio lordo e netto sul capitale impegnato (spot + margine del perpetual) e quanti mesi completi hanno superato il 2%. Non include la variazione della base, la liquidazione della gamba short né il rischio exchange.
- **Peggior trade OOS:** nella tabella delle soglie, la nuova colonna mostra la perdita peggiore del campione di verifica. Per simulare "TP senza SL" imposta "Margine prima dell’uscita in perdita" a 99: resta solo l'uscita a tempo, e la colonna mostra il rischio che il TP nasconde.
