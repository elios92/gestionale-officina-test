# Specifica preliminare: GitHub Releases per GDO Bike

> **Stato:** progettazione. Questo repository non contiene attualmente il progetto Android completo né un artefatto APK verificato. La presente documentazione non attiva aggiornamenti nell'app.

## 1. Canale e visibilità

Questo repository è **pubblico**. Chiunque potrebbe scaricare gli asset di una Release pubblicata qui. Prima della prima pubblicazione il proprietario deve autorizzare esplicitamente il canale, la visibilità e l'APK specifico.

La sorgente di release stabili, **solo dopo l'attivazione**, può essere:
`GET https://api.github.com/repos/elios92/gestionale-officina-test/releases/latest`

Questa API fornisce l'ultima release **pubblicata e non prerelease**, non garantisce che ne esista una: trattare 404 e errori HTTP come **controllo non riuscito/non configurato**, non come “app aggiornata”.

Per canali di collaudo separati: gestire esplicitamente release prerelease, senza esporle agli utenti del canale stabile e senza sovrapporre `versionCode`.

**Non inserire token GitHub personali nell'APK.** Repository privati richiedono una strategia autenticata lato server; GitHub Releases pubbliche sono scaricabili da chiunque. Rispettare le limitazioni di API, rete e uso del servizio.

## 2. Regole di pubblicazione di una release

Prima di pubblicare una release reale:

1. Confermare repository, branch, canale, autorizzazione del titolare e provenienza della build.
2. Verificare il progetto sorgente reale e sincronizzarlo **senza** sovrascrivere o cancellare file esistenti; usare un repository privato se il codice non deve essere pubblico.
3. Compilare e testare l'APK da un commit identificabile; non pubblicare il solo output debug di Android Studio.
4. Usare una configurazione di **firma release** gestita in sicurezza, con chiavi fuori dal repository. La firma deve essere compatibile con le installazioni da aggiornare, incluso l'`applicationId`.
5. Verificare `versionName`, `versionCode` strettamente crescente, `minSdk`, eventuali split/ABI e compatibilità dispositivi.
6. Pubblicare tag `vX.Y.Z`, note di rilascio verificabili, APK firmato, digest **SHA-256** del file e riferimenti al commit/build. Il digest non sostituisce la verifica della firma Android.
7. Escludere dati clienti, database, backup, configurazioni private, token, `google-services.json` privati, keystore e password.
8. Solo dopo collaudo sul tablet e autorizzazione esplicita rendere la Release accessibile al canale previsto.

**Non esistono in questo documento URL di APK, release/versioni fittizie né workflow di firma funzionanti.**

## 3. Collegamento all'app Android

Implementare sul progetto Android **dopo averlo realmente aperto e verificato**:

- Sorgente remota e parser GitHub Release con errori HTTP, timeout, rate limit, offline e JSON inatteso gestiti.
- Confronto affidabile fra la versione installata e un `versionCode` dichiarato in metadati attendibili della release; **non dedurlo ciecamente dal tag vX.Y.Z**.
- Associazione fra asset, `applicationId`, canale, versione, compatibilità e build firmata; evitare downgrade e notifiche di APK incompatibili.
- Aggiornamento di stato UI soltanto dopo controllo remoto riuscito. In caso contrario: “Controllo non disponibile” o errore corrispondente.
- `WorkManager` periodico (ad es. 24 ore, **non orario garantito**) e verifica anche al rientro in app; nessuna promessa di avvisi immediati.
- Canale di notifica e richiesta `POST_NOTIFICATIONS` su Android 13+, preferenze utente e deduplicazione per release già vista.
- L'app normalmente chiusa può ricevere lavori pianificati; un **force stop** impedisce le normali esecuzioni fino al riavvio manuale.
- Notifica che apre le informazioni di rilascio tramite destinazione verificata. L'installazione deve seguire le regole Android/Google Play applicabili, con consenso esplicito e validazione della firma, non avvenire in silenzio.
- Se l'app è distribuita su Google Play, verificare la compatibilità con il sistema di aggiornamenti del Play Store e non sostituirlo arbitrariamente con sideload.

La verifica delle nuove **librerie di sviluppo** (Gradle, Room, AndroidX, ecc.) è un sistema separato di CI/repository: notifica lo sviluppatore, **non** il tablet e **non** modifica automaticamente dipendenze o codice.

## 4. Test di accettazione

Su build e fonte di prova **vere**, senza dati del proprietario:

- Nessuna release: risposta esplicita “non configurato/non disponibile”; nessun falso “aggiornato”.
- Release della stessa versione: nessuna notifica.
- Release autenticata con `versionCode` superiore: nuova versione rilevata.
- Release prerelease ignorata dal canale stabile.
- Release con metadata mancanti, firma incompatibile o asset assente: nessuna proposta installabile.
- Rete assente, HTTP 403/404/429/5xx: errori corretti, nessun PASS fittizio.
- Worker davvero pianificato, prova Android con app chiusa normalmente, permesso notifiche concesso/negato, deduplicazione, tocco della notifica e riavvio dispositivo.
- Nessun dato Room, wizard, menu, magazzino, licenza o file utente alterato.
- Test funzionali reali e report PASS/FAIL/NON ESEGUITO distinti; una notifica sintetica manuale **non prova** il controllo online.

## 5. Attività ancora da autorizzare

- Scelta definitiva: distribuzione privata o pubblica, e quale repository ospiterà gli APK firmati.
- Accesso al codice locale aggiornato del progetto Android per sincronizzazione e integrazione.
- Credenziali/chiavi di firma attraverso meccanismi sicuri (mai in chat, commit o app).
- Prima release effettiva e distribuzione.
- Eventuale CI GitHub Actions solo dopo verifica del codice sorgente, della privacy e della firma.

**Questo file è solo una specifica e non certifica un sistema di aggiornamenti operativo.**
