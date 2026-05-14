# Accesso alla chat di Viber

## Il DB principale è Contacts.data, dove troviamo le chat in chiaro che possiamo visualizzare.
#### Contacts.Data si trova in /var/mobile/Containers/Shared/AppGroup/<group.viber.share.container>/com.viber/database

### La tabella ZCONVERSATION contiene ogni singola chat. Tutti i messaggi di ogni chat sono contenuti nella tabella ZVIBERMESSAGE.


## ZVIBERMESSAGE

I principali campi che ci servono per recuperare i messaggi di una singola chat sono:

**ZCONVERSATION** indica il valore del campo Z_PK della tabella [ZCONVERSATION](#ZCONVERSATION)

**ZDATE** = timestamp del messaggio

**ZPHONENUMINDEX** = il valore del campo Z_PK della tabella [ZPHONENUMBER](#ZPHONENUMBER), se è NULL vuol dire che il messaggio è stato inviato dal target.

**ZCALLTYPE** = valorizzato se si tratta di una chiamata. I possibili valori sono: **incoming**, **outgoing**, **incoming_viber_with_video**, **outgoing_viber_with_video**, **missed**,...

**ZGALLERYTYPE** = valorizzato se l'attachment è di tipo picture.

**ZLOCATION** = valorizzato se il messaggio è di tipo positioning, indica il valore del campo Z_PK della tabella [ZVIBERLOCATION](#ZVIBERLOCATION)

**ZTEXT** con campo **ZSYSTEMTYPE**  a NULL, vuol dire che il valore di ZTEXT è il testo della chat \
**ZTEXT** con campo **ZSYSTEMTYPE** a formatted, vuol dire che il valore di ZTEXT è un Json e si tratta di un  Contatto condiviso in chat.

**ZATTACHMENT** = il valore del campo Z_PK della tabella [ZATTACHMENT](#ZATTACHMENT)


## ZCONVERSATION

I principali campi che ci servono per recuperare le info di una chat sono:

**ZINTERLOCUTOR** = valorizzato se si tratta di una chat non di gruppo, indica il valore del campo Z_PK della tabella [ZMEMBER](#ZMEMBER) e della tabella [ZPHONENUMBER](#ZPHONENUMBER)

**ZNAME** = valorizzato se si tratta di un gruppo, indica i nomi dei partecipanti ad un gruppo.

**ZGROUPID** = valorizzato se si tratta di un gruppo

## ZMEMBER

Da questa tabella recuperiamo i nomi dei contatti da questi campi:

**ZDISPLAYFULLNAME**, **ZDISPLAYSHORTNAME**, **ZNAME**

## ZPHONENUMBER

Da questa tabella recuperiamo i numeri dei contatti da questi campi:

**ZCANONIZEDPHONENUM**, **ZPHONE**

## ZATTACHMENT

Da questa tabella recuperiamo le informazioni degli attachments, in particolar modo picture, file audio, file, video (per la location non bisogna accedere a questa tabella).

**ZTYPE** = indica il tipo di attachments che può essere. I possibili valori sono picture, audio, customLocation, file, video

**ZNAME** = filename dell'attachment. 
Tutti gli attachments risiedono nella folder Documents dell'app
 */var/mobile/Containers/Data/Application/(identificativorandomicoapp)/Documents*

 In particolar modo \
 **Video** e **Picture** li troviamo  nella folder *Attachments* \
 **File** nella folder *FileMessages* \
 **Audio** nella folder *VoiceMessages*

**ZFILESIZE** = indica la size dell'attachment.

## ZVIBERLOCATION

Da questa tabella recuperiamo le informazioni delle positiong condivise:

**ZADDRESS** = indica l'indirizzo completo della positioning.

**ZLATITUDE** = indica la latitudine della positioning.

**ZLONGITUDE** = indica la longitudine della positioning.

#### Di sicuro oltre queste informazioni, che ci consentono di mostrare le chat viber, ci sono tantissime altre informazioni che si possono reperire da altri DB presenti in Viber e anche dallo stesso DB (Contacts.data) che abbiamo ora analizzato.

