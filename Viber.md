# Viber iOS DB Reconstruction

## Obiettivo

Ricostruire una chat Viber partendo dal database locale iOS.

---

# 1. Database da analizzare

Percorso iOS:

```text
/var/mobile/Containers/Shared/AppGroup/<group.viber.share.container>/com.viber/database/Contacts.data
```

Le tabelle utili sono:

```text
ZVIBERMESSAGE
ZCONVERSATION
ZMEMBER
ZPHONENUMBER
ZATTACHMENT
ZVIBERLOCATION
```

---

# 2. Tabella `ZVIBERMESSAGE`

```text
La tabella ZVIBERMESSAGE contiene tutti i messaggi di ogni chat.
```
I principali campi che ci servono per recuperare i messaggi di una singola chat sono:

| Campo | A cosa serve |
|---|---|
| `ZCONVERSATION` | indica il valore del campo Z_PK della tabella [ZCONVERSATION](#3)
| `ZDATE` | timestamp del messaggio
| `ZPHONENUMINDEX` | il valore del campo Z_PK della tabella [ZPHONENUMBER](#ZPHONENUMBER), se è NULL vuol dire che il messaggio è stato inviato dal target.
| `ZCALLTYPE` | valorizzato se si tratta di una chiamata. I possibili valori sono: incoming, outgoing, incoming_viber_with_video, outgoing_viber_with_video, missed
| `ZGALLERYTYPE` | valorizzato se l'attachment è di tipo picture.
| `ZLOCATION` | valorizzato se il messaggio è di tipo positioning, indica il valore del campo Z_PK della tabella ZVIBERLOCATION
| `ZTEXT` | quando il campo  `ZSYSTEMTYPE` è NULL, il suo valore indica il testo della chat, mentre quando `ZSYSTEMTYPE` è valorizzato con formatted, è valorizzato con un Json da dove recuperiamo il Contatto condiviso in chat.
| `ZSYSTEMTYPE` | se valorizzato con "formatted" significa che in `ZTEXT` troveremo un json con il Contatto condiviso in chat.
| `ZATTACHMENT` |  il valore del campo Z_PK della tabella [ZATTACHMENT](#ZATTACHMENT)

---

# 3. Tabella `ZCONVERSATION`
```text
La tabella ZCONVERSATION contiene ogni singola chat.
```
I principali campi che ci servono per recuperare le info di una chat sono:

| Campo | A cosa serve |
|---|---|
| `ZINTERLOCUTOR` | valorizzato se si tratta di una chat non di gruppo, indica il valore del campo Z_PK della tabella [ZMEMBER](#ZMEMBER) e della tabella [ZPHONENUMBER](#ZPHONENUMBER)
| `ZNAME` | valorizzato se si tratta di un gruppo, indica i nomi dei partecipanti ad un gruppo.
| `ZGROUPID` | valorizzato se si tratta di un gruppo.

---

# 4. Tabella `ZMEMBER`
```text
La tabella ZMEMBER contiene informazioni di ogni contatto.
```
I principali campi sono:

| Campo | A cosa serve |
|---|---|
| `ZDISPLAYFULLNAME` | contiene il nome completo visualizzato del contatto Viber.
| `ZDISPLAYSHORTNAME` | contiene il nome visualizzato in versione short del contatto Viber.
| `ZNAME` | contiene il nome del contatto Viber salvato nella rubrica del tefono.

---

# 4. Tabella `ZPHONENUMBER`
```text
La tabella ZPHONENUMBER contiene informazioni sui numeri di telefono di ogni contatto.
```
I principali campi sono:

| Campo | A cosa serve |
|---|---|
| `ZCANONIZEDPHONENUM` | contiene il numero di telefono completo e formattato del contatto Viber.
| `ZPHONE` | contiene il nome visualizzato in versione short del contatto Viber.

---

# 5. Tabella `ZATTACHMENT`
```text
La tabella ZATTACHMENT contiene informazioni sugli attachments. 
In particolar modo picture, file, audio, video. Per la location non recupereremo informazioni da questa tabella.
```

I principali campi sono:

| Campo | A cosa serve |
|---|---|
| `ZTYPE` | indica il tipo di attachments che può essere. I possibili valori sono picture, audio, customLocation, file, video.
| `ZNAME` | filename dell'attachment. 
| `ZFILESIZE` | indica la size dell'attachment.

 Tutti gli attachments risiedono nella folder Documents dell'app `/var/mobile/Containers/Data/Application/(identificativorandomicoapp)/Documents`
```text
 In particolar modo
 Video e Picture li troviamo nella folder Attachments
 File nella folder FileMessages
 Audio nella folder VoiceMessages
```
---

# 6. Tabella `ZVIBERLOCATION`
```text
La tabella ZVIBERLOCATION contiene informazioni sulle positioning condivise in chat.
```
I principali campi sono:

| Campo | A cosa serve |
|---|---|
| `ZADDRESS` | indica l'indirizzo completo della positioning.
| `ZLATITUDE` | indica la latitudine della positioning. 
| `ZLONGITUDE` | indica la longitudine della positioning.

---

# 7. Conclusioni
```text
Di sicuro oltre queste informazioni, che ci consentono di mostrare le chat Viber, ci sono tantissime altre informazioni che si possono reperire da altri DB presenti in Viber e anche dallo stesso DB (`Contacts.data`) che abbiamo ora analizzato.
```





---
