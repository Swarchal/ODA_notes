# Harmony ODA API

The aim of this is to figure out how to fetch metadata from Harmony via the ODA api.
This might just be notes, might have scripts once I've figured more of it out.

PerkinElmer/Revvity supply no documentation on using the ODA (Opera database API).
It looks like Harmony is sending API calls over HTTP, so it is possible to
inspect how Harmony uses the ODA by inspecting the network traffic on a Harmony
computer with WireShark or similar.

Some of the requests are authenticated with a Session ID. I don't know how
these Session IDs are generated or how long they are valid for. In theory it
should be possible to sniff the Session ID and then send API calls query the
ODA through an external script.

## What I've found so far:

### Fetching images
Same as in the indexfile.
This doesn't require  authentication.
```http
GET /ODA/Images/C/{signature}/{imgID?}.tiff
```

```
User-Agent: Acapella/5.2\r\n
```

### Fetching image analysis metadata
```http
GET /ODA/XML/{signature}.xml
```

This header is present in the captured GET requests, but seems to work without:
```
Cookie: ASP.NET_SessionId={sessionID}\r\n
User-Agent: OdaClient\r\n
```

This returns a Harmony Acapella text (non-standard xml) file. Many of the
strings as zlib base64 encoded. Seems to be missing a lot of basic metadata
such as the analysis name and timestamps, though it's difficult to grep for
anything with them being encoded.

### Fetching ???
It looks like it first makes multiple POST requests to
```http
POST /ODA/OdaService.asmx
```
before any get requests with a `SessionID`. The POST requests contain the
signature etc. I guess the session ID is used to track which signature is
relevant for any get requests based on the `SessionID` rather than the signature.

```http
GET /ODA/S({SessionID})/Buffers/ObjectBaseData
```

```http
GET /ODA/S({SessionID})/Buffers/ObjectDisplayData
```

```http
GET /ODA/S({SessionID})/Buffers/Keywords
```

```http
GET /ODA/S({SessionID})/Buffers/KeywordValues
```

```http
GET /ODA/S({SessionID})/Buffers/KWBNextLevel
```

```http
GET /ODA/S({SessionID})/Buffers/KeywordTypeDescriptions
```

```http
GET /ODA/S({SessionID})/Buffers/GetComment
```

```
Cookie: ASP.NET_SessionId={sessionID}\r\n
User-Agent: OdaClient\r\n
```

### Posting ???
Including a header containing a session ID as a cookie.
```http
POST /ODA/OdaService.asmx
```

POST requests to `/ODA/OdaService.asmx` appears to be required for later GET
requests, which only contain the `SessionID`.
