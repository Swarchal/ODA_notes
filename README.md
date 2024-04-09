# Harmony ODA API

PerkinElmer/Revvity supply no documentation on using the ODA (Opera database API).
It looks like Harmony is sending API calls over HTTP, so it is possible to
inspect how Harmony uses the ODA by inspecting the network traffic on a Harmony
session.

It seems to work via SOAP, sending XML bodies with requests for data. Quite
often the response is then a URL to second as a second request which
returns the requested data, again as XML.


## Generating a session ID

Many of the requests require a session ID. You can generated your own
valid session ID through a post request to `ODA/ODAService.xml`.

```python
header = {"Content-Type": "text/xml; charset=utf8"}
body = """
    <s:Envelope
      xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
      <s:Body>
        <GetSessionID
          xmlns="http://www.perkinelmer.com/PEHH.ODA"
          xmlns:a="http://schemas.datacontract.org/2004/07/PEHH.OdaClient"
          xmlns:i="http://www.w3.org/2001/XMLSchema-instance" />
      </s:Body>
    </s:Envelope>
"""
response = requests.post(url=f"{HOST}/ODA/ODAService.asmx", headers=headers, data=body)

# parse session ID from response text or header
```

The session ID is then contained within the returned xml, as well as in the headers
under "Set-Cookie".

The session ID seems remain valid for at least several days.


## Logging in

A number of metadata requests require a logged in user. This can be done after
obtaining a session ID.

```python
headers = {
    "Content-Type": "text/xml; charset=utf-8",
    "Cookie": f"ASP.NET_SessionId={session_id}",
}
body = f"""
    <s:Envelope
    xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
      <s:Body>
        <Login
          xmlns="http://www.perkinelmer.com/PEHH.ODA"
          xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
            <user>{user}</user>
            <passwd>{password}</passwd>
        </Login>
      </s:Body>
    </s:Envelope>
"""
return requests.post(
    url=f"{HOST}/ODA/OdaService.asmx", headers=headers, data=body
)
```
