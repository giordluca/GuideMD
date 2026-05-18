# IMO iOS DB Reconstruction

## Objective

Rebuild a Imo chat from the local iOS database.

---

# 1. Database to analyze

Path iOS:

```text
/var/mobile/Containers/Data/Application/<UUID>/Documents/IMODb2.sqlite
```

Useful tables are:

```text
ZIMOCONTACT
ZIMOCHATMSG
```

---

# 2. Table `ZIMOCONTACT`

```text
The ZIMOCONTACT table contains all the information of the contacts and also of the group chats
```
The main fields are:

| Field | Description |
|---|---|
| `ZBUID` | contact ID, also present for the target. A group chat also has a ZBUID.
| `ZALIAS` | this represents the contact's name for the imo app; it's also used for targeting. It's not used for group chat.
| `ZDISPLAY` | represents the name displayed in imo. For group chats, it is filled with the participants' imo names.
| `ZPH_NAME`, `ZREMARKNAME`  | represent the contact's name
| `ZPHONE` | unformatted telephone number, without the international prefix.
| `ZDIGIT_PHONE` | formatted telephone number, without the international prefix
| `ZFORMATED_PHONE` | formatted telephone number, complete with international prefix.


---

# 3. Table `ZIMOCHATMSG`
```text
The ZIMOCHATMSG table is the main table, from where we retrieve all messages.
```

The main fields are:

| Field | Description |
|---|---|
| `ZTS` | message timestamp.
| `ZA_UID` | ZBUID of the ZIMOCONTACT table, indicates who sent the message, is NULL if the message was sent by the target.
| `ZBUID` | ZBUID of the ZIMOCONTACT table, indicates who the target is chatting with.
| `ZTEXT` | valued only for some message types.
| `ZIMDATA` | A blob containing all the information about the message, including the information already in the table, including the timestamp. It's a .bin file.

---

# 4. Field `ZIMDATA`

```text
The value of the ZIMDATA field is a Blob. To read it from a Mac, we convert this .bin file to text with the plutil command.

Example: plutil -p Blob.bin > Blob

From this, we'll be able to understand what type of message it is:
text, photo, video, call, file, audio, positioning
and what information to find that isn't found in the ZIMOCHATMSG table.
```
Now let's analyze this complex file type. \
Every Blob is an NSKeyedArchiver file.

The main keys are:

| Key | Description |
|---|---|
| `$top` | we'll understand the object where we should start our search.
| `$objects` | This is the main array that provides all the information. For ease of reading, we'll call it `arrayObjects`.

As values ​​of `arrayObjects`, in addition to the usual data types we will also find NSDictionaries which are made up of arrays of keys, `NS.keys`, and the corresponding `NS.objects` values.

```text
Keys and values ​​are read in parallel:
NS.keys[0] → NS.objects[0]
NS.keys[1] → NS.objects[1]
```

Both as keys and as values ​​we will find `CFKeyedArchiverUID` objects
```text
<CFKeyedArchiverUID 0x6000030680e0 [0x20dc20120]>{value = 1}
```
This means that this object points to index 1 of our `arrayObjects`

## 4.1 Retrieve the object from where to start our search

```text
"$top" => {
    "root" => <CFKeyedArchiverUID 0x6000030680e0 [0x20dc20120]>{value = 1}
}
```
So we will start our search from the object we will find at index 1 of `arrayObjects`

## 4.2 Understanding the type of Blob we are analyzing

We retrieve the indices of the `type` and `objects` values ​​from `arrayObjects`. \
`type` is always present. \
`objects` is not present for some Blob types.

We check if `objects` exists, if not, the value of the `type` index will tell us what type of Blob we are analyzing.
If `objects` is present, we retrieve the object that identifies it and search for the `type` value within it, starting from the index retrieved previously.

```text
{
  "$archiver" => "NSKeyedArchiver"
  "$objects" => [
    .
    1 => {
      "$class" => <CFKeyedArchiverUID 0x600003068400 [0x20dc20120]>{value = 32}
      "NS.keys" => [
        0 => <CFKeyedArchiverUID 0x600003068180 [0x20dc20120]>{value = 2}
        1 => <CFKeyedArchiverUID 0x6000030681a0 [0x20dc20120]>{value = 3}
        2 => <CFKeyedArchiverUID 0x6000030681c0 [0x20dc20120]>{value = 4}
        .
        5 => <CFKeyedArchiverUID 0x600003068220 [0x20dc20120]>{value = 7}    
        6 => <CFKeyedArchiverUID 0x600003068240 [0x20dc20120]>{value = 8}
        .
        .
      ]
      "NS.objects" => [
        0 => <CFKeyedArchiverUID 0x6000030682c0 [0x20dc20120]>{value = 12}
        1 => <CFKeyedArchiverUID 0x6000030682e0 [0x20dc20120]>{value = 13}
        2 => <CFKeyedArchiverUID 0x600003068300 [0x20dc20120]>{value = 14}
        .
        .
        6 => <CFKeyedArchiverUID 0x600003068380 [0x20dc20120]>{value = 18}
        .
        .
      ]
    }
    2 => "file_name"
    3 => "type"
    4 => "file_size"
    5 => "contact_note"
    6 => "file_task_status"
    7 => "url"
    8 => "objects"
    .
    .
    .
    12 => "ip.rtf"
    13 => "bigo_uploaded"
    14 => 745
    .
    17 => "http://imfile.imostatic.com/eu_live/suf/M08/17/78/H7PHAGoDSgaEevFfAAAAAEfnL1Y294.rtf?autoIdx=1&crc=1206333270&offset=0&size=745&auth-token=vMn64rzB6zpX9BTXe-aSk7ouSAi0pNKU12iPtgY0E1I:eyJleHBpcmUiOjE3ODYzNzY0NTQsImFwcGlkIjo2Miwic2VydmljZS1uYW1lIjoiUElOIiwiY3VzdG9tMSI6InNlbmRfaW1fbm9fY2hlY2sifQ"
    18 => {
      "$class" => <CFKeyedArchiverUID 0x6000030684e0 [0x20dc20120]>{value = 40}
      "NS.objects" => [
        0 => <CFKeyedArchiverUID 0x6000030684c0 [0x20dc20120]>{value = 19}
      ]
    }
    19 => {
      "$class" => <CFKeyedArchiverUID 0x600003068400 [0x20dc20120]>{value = 32}
      "NS.keys" => [
        0 => <CFKeyedArchiverUID 0x600003068500 [0x20dc20120]>{value = 20}
        1 => <CFKeyedArchiverUID 0x600003068520 [0x20dc20120]>{value = 21}
        2 => <CFKeyedArchiverUID 0x600003068540 [0x20dc20120]>{value = 22}
        3 => <CFKeyedArchiverUID 0x600003068560 [0x20dc20120]>{value = 23}
        4 => <CFKeyedArchiverUID 0x600003068580 [0x20dc20120]>{value = 24}
        5 => <CFKeyedArchiverUID 0x6000030685a0 [0x20dc20120]>{value = 25}
        6 => <CFKeyedArchiverUID 0x6000030685c0 [0x20dc20120]>{value = 26}
        7 => <CFKeyedArchiverUID 0x6000030685e0 [0x20dc20120]>{value = 27}
        8 => <CFKeyedArchiverUID 0x6000030681a0 [0x20dc20120]>{value = 3}
      ]
      "NS.objects" => [
        0 => <CFKeyedArchiverUID 0x600003068300 [0x20dc20120]>{value = 14}
        1 => <CFKeyedArchiverUID 0x600003068600 [0x20dc20120]>{value = 28}
        2 => <CFKeyedArchiverUID 0x600003068620 [0x20dc20120]>{value = 33}
        3 => <CFKeyedArchiverUID 0x600003068640 [0x20dc20120]>{value = 34}
        4 => <CFKeyedArchiverUID 0x600003068660 [0x20dc20120]>{value = 36}
        5 => <CFKeyedArchiverUID 0x600003068680 [0x20dc20120]>{value = 37}
        6 => <CFKeyedArchiverUID 0x6000030686a0 [0x20dc20120]>{value = 38}
        7 => <CFKeyedArchiverUID 0x6000030686c0 [0x20dc20120]>{value = 39}
        8 => <CFKeyedArchiverUID 0x6000030686e0 [0x20dc20120]>{value = 31}
      ]
    }
    .
    .
    .
    31 => "file"
    .
  ]
  "$top" => {
    "root" => <CFKeyedArchiverUID 0x6000030680e0 [0x20dc20120]>{value = 1}
  }
  "$version" => 100000
}
```

The value `objects` is found at index 8, while `type` is found at index 3.
Starting from the object at index 1 that the key `$top` indicated, the key with value=8 corresponds to the object with value=18.

Object 18 has a single object that takes us to index 19. \
At index 19, we find the `objects` object, and it's here that we look for our `type`, which has index 3. \
So, we look in the keys for value = 3, which corresponds to \
<CFKeyedArchiverUID 0x6000030686e0 [0x20dc20120]>{value = 31}. \
Go to index 31 of `arrayObjects` and find the type of our Blob. \
In this case, it's "file."

This is the basic procedure for all message types. \
Now, for each type of Blob, we will show where to retrieve the most important information that we will not find in the ZIMOCHATMSG table.

## 4.3 Blob type FILE
```text
For this type of file message, the "type" could be either "file" or "bigo_uploaded."
```
The main information we can retrieve is:

| Key | Description | Where we find it |
|---|---|---|
| `url` | indicates the path of the resource | object of `$top`
| `filesize` | file size | object of `$top`
| `file_name` | filename | value object `objects`
| `mime` | myme type | value object `type_specific_params`

[BlobTypeFile](Files/BlobTypeFile)

## 4.4 Blob type TEXT
```text
For a text message, we'll use "im" as the type.
```
We don't retrieve anything from this blob, since we retrieve the text from the `ZTEXT` field of the `ZIMOCHATMSG` table.

[BlobTypeText](Files/BlobTypeText)

## 4.5 Blob type POSITIONING
```text
For a positioning message, we will use the type "location".
```
From this blob we can retrieve all the information regarding the sent location, while from the `ZLOCATIONIMAGEURL` field of the `ZIMOCHATMSG` table we retrieve the path to the photo of the location.

The main information we can retrieve is:

 Key | Description | Where we find it |
|---|---|---|
| `latitude` | latitude | object of `$top`
| `longitude` | longitude | object of `$top`
| `placeName` | address without the city | object of `$top`
| `address` | address with the city | object of `$top`

[BlobTypePositioning](Files/BlobTypePositioning)

## 4.6 Blob type CALL
```text
For a call type message we will find "call_log" as type.
```

The main information we can retrieve is:

 Key | Description | Where we find it |
|---|---|---|
| `chat_type` | indicates whether it is "audio_chat" or "video_chat" | object of `$top`
| `call_duration` | indicates the duration of the call in seconds | object of `$top`

[BlobTypeCall](Files/BlobTypeCall)

## 4.7 Blob type AUDIO
```text
For a audio type message we will find "audio" as type.
```

The main information we can retrieve is:

| Key | Description | Where we find it |
|---|---|---|
| `duration` | duration audio | value object `type_specific_params`
| `filename` | file size | value object `objects`
| `file_name` | filename | value object `objects`
| `mime` | myme type | value object `type_specific_params`

The resource resides in the app's `Library/Caches/videos` folder.

[BlobTypeAudio](Files/BlobTypeAudio)

## 4.7 Blob type VIDEO
```text
For a video type message we will find "video" as type.
```

The main information we can retrieve is:

| Key | Description | Where we find it |
|---|---|---|
| `duration` | duration video | value object `type_specific_params`
| `bigo_url` | url video | value object `objects`
| `thumbnailUrl` | url thumbnail | value object `type_specific_params`

[BlobTypeVideo](Files/BlobTypeVideo)

## 4.7 Blob type PHOTO

From this blob we can retrieve all the information regarding all types of images, whether shared, taken, or taken from the device.

```text
For a photo type message we will find "image" as type.
```

| Key | Description | Where we find it |
|---|---|---|
| `filename` | name image | value object `objects`
| `filesize` | size image| value object `objects`
| `original_height` | height image| value object `type_specific_params`
| `original_width` | width image| value object `type_specific_params`
| `mime` | mime type | value object `type_specific_params`
| `bigo_url` | url image| value object `objects`
| `object_id` | filename | value object `objects`

To retrieve the resource we first check the `bigo_url` and then if it is not there we retrieve `object_id`.

`object_id` is a hidden file to append "|?size_type=webp&fit=1.png"
```text
Example if the value of "object_id" is .K1PCFLVaKGSgIoghcmCISHZur0p
the full name will be .K1PCFLVaKGSgIoghcmCISHZur0p|?size_type=webp&fit=1.png
```

The resource resides in the app's `Library/Caches/videos` folder.

[BlobTypePhoto](Files/BlobTypePhoto)


## 5. Conclusions

In addition to this information,
we can retrieve a lot more from both the "IMODb2.sqlite" database and other databases in IMO app.

[IMODb2.sqlite](Files/IMODb2.sqlite)
