# Viber iOS DB Reconstruction

## Objective

Rebuild a Viber chat from the local iOS database.

---

# 1. Database to analyze

Path iOS:

```text
/var/mobile/Containers/Shared/AppGroup/<group.viber.share.container>/com.viber/database/Contacts.data
```

Useful tables are:

```text
ZVIBERMESSAGE
ZCONVERSATION
ZMEMBER
ZPHONENUMBER
ZATTACHMENT
ZVIBERLOCATION
```

---

# 2. Table `ZVIBERMESSAGE`

```text
The ZVIBERMESSAGE table contains all the messages from each chat.
```
The main fields we need to retrieve messages from a single chat are:

| Field | Description |
|---|---|
| `ZCONVERSATION` | indicates the value of the Z_PK field of the [ZCONVERSATION](#3-table-zconversation) table 
| `ZDATE` | message timestamp
| `ZPHONENUMINDEX` | the value of the Z_PK field of the [ZPHONENUMBER](#5-table-zphonenumber) table, if it is NULL means that the message was sent by the target.
| `ZCALLTYPE` | valued if it is a call. Possible values ​​are: incoming, outgoing, incoming_viber_with_video, outgoing_viber_with_video, missed
| `ZGALLERYTYPE` | set if the attachment is of type picture.
| `ZLOCATION` | set if the message is of the positioning type, indicates the value of the Z_PK field of the [ZVIBERLOCATION](#7-table-zviberlocation) table
| `ZTEXT` | when the `ZSYSTEMTYPE` field is NULL, its value indicates the chat text, while when `ZSYSTEMTYPE` is set to "formatted", it is set to a Json from which we retrieve the Contact shared in the chat.
| `ZSYSTEMTYPE` | if set to "formatted" it means that in `ZTEXT` we will find a json with the Contact shared in chat.
| `ZATTACHMENT` | the value of the Z_PK field of the [ZATTACHMENT](#6-table-zattachment) table

---

# 3. Table `ZCONVERSATION`
```text
The ZCONVERSATION table contains every single chat.
```
The main fields we need to retrieve chat information are:

| Field | Description |
|---|---|
| `ZINTERLOCUTOR` | set to zero if it is a non-group chat, indicates the value of the Z_PK field of the [ZMEMBER](#4-table-zmember) table and the [ZPHONENUMBER](#5-table-zphonenumber) table
| `ZNAME` | valued if it is a group, indicates the names of the participants in a group.
| `ZGROUPID` | valued if it is a group.

---

# 4. Table `ZMEMBER`
```text
The ZMEMBER table contains information about each contact.
```
The main fields are:

| Field | Description |
|---|---|
| `ZDISPLAYFULLNAME` | contains the full display name of the Viber contact.
| `ZDISPLAYSHORTNAME` | contains the shortened display name of the Viber contact.
| `ZNAME` | contains the name of the Viber contact saved in your phone's address book.

---

# 5. Table `ZPHONENUMBER`
```text
The ZPHONENUMBER table contains information about the phone numbers of each contact.
```
The main fields are:

| Field | Description |
|---|---|
| `ZCANONIZEDPHONENUM` | contains the full, formatted phone number of the Viber contact.
| `ZPHONE` | contains the incomplete and unformatted phone number of the Viber contact.

---

# 6. Table `ZATTACHMENT`
```text
The ZATTACHMENT table contains information about attachments.
Especially pictures, files, audio, and video. We won't retrieve location information from this table.
```

The main fields are:

| Field | Description |
|---|---|
| `ZTYPE` | Specifies the type of attachment it can be. Possible values ​​are picture, audio, customLocation, file, or video.
| `ZNAME` | filename of the attachment.
| `ZFILESIZE` | indicates the size of the attachment.

All attachments reside in the app's Documents folder `/var/mobile/Containers/Data/Application/<UUID>/Documents`
```text
 Video and Photo can be found in the Attachments folder
 File in the FileMessages folder
 Audio in the VoiceMessages folder
```
---

# 7. Table `ZVIBERLOCATION`
```text
The ZVIBERLOCATION table contains information about the positionings shared in chat.
```
The main fields are:

| Field | Description |
|---|---|
| `ZADDRESS` | indicates the complete address of the positioning.
| `ZLATITUDE` | indicates the latitude of the positioning.
| `ZLONGITUDE` | indicates the longitude of the positioning.

---

# 8. Conclusions
```text
Of course, in addition to this information,
we can retrieve a lot more from both the "Contacts.data" database and other databases in Viber.
The [Contacts.data](./Contacts.data) database is attached.
```







---
