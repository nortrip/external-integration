

# Spec

## Create member webhook

#### POST https://members.nortrip.no/webhooks/external/order_paid

#### Headers:
Each request should contain an Authorization header,
with token type "hmac-sha256", and token content being
the calculated signature of the body.
See [Signature section](#signature).

```
Authorization: hmac-sha256 <signature>
```

#### Body:
```json
{
    "partner_id": "campr",
    "order_id": "consistent order id",
    "regions": ["nor", "swe"],
    "valid_until": "2024-01-01T00:00:00Z",
    "valid_after": "2023-12-01T00:00:00Z",
    "customer": {
        "name": "Name Here",
        "email": "name@gmail.com",
        "locale": "en",
        "regno": "BR12345"
    }
}
```

- `partner_id`: Required. Static value identifying the requesting partner. Use `campr`.
- `order_id`: Optional, but recommended. Any consistent id for a specific order. Supplying it will make the request idempotent.
- `regions`: Required. List of regions to grant access to. Either `nor`, `swe`, or both.
- `valid_until`: Required. When the expiry of the membership should be.
- `valid_after`: Optional. When the membership should start. If not supplied it will start at the time of the request.
- `customer`: Required.
    - `name`: Required. Full name of person.
    - `email`: Required. Contact email of person.
    - `locale`: Optional. Persons preferred language. Supported, one of: `en`, `nb`, `de`, `fr`, `nl`. Defaults to `en`.
    - `regno`: Required. Car registration number for the car their membership should be attached to.

#### Response:
On success, the response will be contain a success status code, and a simple response body containing the assigned membership number. This can be given to the person.

200 OK
```js
{
    "member_number": 112233445566
}
```

#### Signature:
Minify the POST json body, calculate the SHA256 HMAC with Base64 encoding like:
```bash
Base64(
    HMAC(
        SharedSecretKey,
        JsonBody,
        SHA256,
    )
)
```

Example using the example body, example secret key `secret_key_9999`:

```js
let secretKey = "secret_key_9999";
// 'secret_key_9999'
let jsonBody = JSON.stringify({
    "partner_id": "campr",
    "order_id": "consistent order id",
    "regions": ["nor", "swe"],
    "valid_until": "2024-01-01T00:00:00Z",
    "valid_after": "2023-12-01T00:00:00Z",
    "customer": {
        "name": "Name Here",
        "email": "name@gmail.com",
        "locale": "en",
        "regno": "BR12345"
    }
});
// '{"partner_id":"campr","order_id":"consistent order id","regions":["nor","swe"],"valid_until":"2024-01-01T00:00:00Z","valid_after":"2023-12-01T00:00:00Z","customer":{"name":"Name Here","email":"name@gmail.com","locale":"en","regno":"BR12345"}}'
let hmacBytes = crypto.createHmac("sha256", secretKey).update(jsonBody).digest();
// <Buffer bb 40 ce a1 ed 30 50 05 14 c1 76 76 10 77 84 fe 6f 4a 7b ce ac ab ca c6 6b 94 6c 01 d2 3a be f6>
let hmacSignature = hmacBytes.toString("base64");
// 'u0DOoe0wUAUUwXZ2EHeE/m9Ke86sq8rGa5RsAdI6vvY='

/// Better yet, bytes and signature can be combined into one step:
let hmacSignature = crypto.createHmac("sha256", secretKey).update(jsonBody).digest("base64");
// 'u0DOoe0wUAUUwXZ2EHeE/m9Ke86sq8rGa5RsAdI6vvY='
```

So header value becomes:

```
Authorization: hmac-sha256 u0DOoe0wUAUUwXZ2EHeE/m9Ke86sq8rGa5RsAdI6vvY=
```

## Open automatic account registration in app

Launch URL: `https://members.nortrip.no/signup?num=<member_number>`

Example:
```swift
UIApplication.shared.open(URL(string: "https://members.nortrip.no/signup?num=112233445566")!)
```

If the app is already installed, this will automatically open the app where
the person can complete their Nortrip account setup.

If the app is not installed, you can query for it before opening the url and
suggest the user download the app, otherwise it'll open the normal Nortrip web interface
which also suggests downloading the app.

