# GNet Quick Start Guide

The GNet platform is an integration platform that connects ground transportation companies, dispatch/reservation systems, and other custom integrations. Members of GNet can send (farm-out) and/or receive (farm-in) reservations. In this process, the sending party (also known as the requester) and the receiving party (the provider) are key players in ensuring smooth reservation transactions. For all API calls, you need to get the token using your API Gateway user.

## Sign Up to GNET
- Visit and signup [https://gnet.grdd.net/registration/](https://gnet.grdd.net/registration/)
- The GNET team will review and approve your account and provide the API credentials.

## Farm In
Create a REST Endpoint to accept POST with the following spec:

- Header: `content-type: application/json`
- Authorization: basic authentication (api_key/api_secret)
- Body: `{reservation json payload: see sample)`

On Successful farm in, please return the following sample payload:
```json
{
  "success": true,
  "reservationId": "11545-001",
  "fees": [],
  "totalAmount": "130.67",
  "transactionId": "cdbfab07-d532-4bfe-b048-6954d94af4e51"
}
```

### Notes:
- Fee array is optional.
- `totalAmount` includes all fees such as tax, VAT, and surcharges.
- We call this endpoint FarmIn Adapter.

On a failed farm in, please return the following sample payload:
```json
{
  "success": false,
  "message": "Unable to process the request due to XYZ.",
  "transactionId": "cdbfab07-d532-4bfe-b048-6954d94af4e51"
}
```

[https://gnet.grdd.net/registration/](https://gnet.grdd.net/registration/)
[https://raw.githubusercontent.com/GRiDD/GRiDD.github.io/main/gnet-sample-reservation.json](https://raw.githubusercontent.com/GRiDD/GRiDD.github.io/main/gnet-sample-reservation.json)

## Reservation Updates
You need to send the reservation updates such as confirmation, status changes like EN_ROUTE, AT_LOCATION to this endpoint. Here is the detailed documentation:
[https://gnet.grdd.net/Platform.svc/providerUpdateStatusByTransactionId/{transactionId}/v1](https://gnet.grdd.net/Platform.svc/providerUpdateStatusByTransactionId/%7BtransactionId%7D/v1)

## Location Updates
You can update the current location of drivers with the following URL:
[https://gnet.grdd.net:9033/ggps.svc/help/operations/saveGPScache](https://gnet.grdd.net:9033/ggps.svc/help/operations/saveGPScache)

### Required fields are:
- `griddid` (your GNet Id)
- `internalBookingId` (your reservation #)
- `latitude`
- `longitude`

## Quote Requests
Check if the payload contains `reservationType` and set it to `QUOTE`.

Full API documentation: [https://gnet.docs.apiary.io/](https://gnet.docs.apiary.io/)

[https://gnet.docs.apiary.io/#/reference/0/provider-system-callback-by-transaction-id/provider-update-status-by-transaction-id/200?mc=reference%2F0%2Fprovider-system-callback-by-transaction-id%2Fprovider-update-status-by-transaction-id%2F200](https://gnet.docs.apiary.io/#/reference/0/provider-system-callback-by-transaction-id/provider-update-status-by-transaction-id/200?mc=reference%2F0%2Fprovider-system-callback-by-transaction-id%2Fprovider-update-status-by-transaction-id%2F200)
[https://gnet.grdd.net/Platform.svc/providerUpdateStatusByTransactionId/%7BtransactionId%7D/v1](https://gnet.grdd.net/Platform.svc/providerUpdateStatusByTransactionId/%7BtransactionId%7D/v1)
[https://gnet.grdd.net:9033/ggps.svc/help/operations/saveGPScache](https://gnet.grdd.net:9033/ggps.svc/help/operations/saveGPScache)

## Farmout
1. Get Token
- POST [https://gnet.grdd.net/Platform.svc/getToken2](https://gnet.grdd.net/Platform.svc/getToken2)
```json
{
  "pw":"<password>",
  "uid":"<user_id>"
}
```

2. Send Reservation
- POST your fully loaded reservation JSON request to:
[https://gnet.grdd.net/Platform.svc/sendTrip/v1](https://gnet.grdd.net/Platform.svc/sendTrip/v1)
- Headers:
  - `token` (from step 1)

3. Register a callback webhook
- Your webhook receives reservation updates from the provider.
- Your endpoint should accept POST and return the 200 status code.

Here is a sample webhook payload you will be receiving:
[https://raw.githubusercontent.com/GRiDD/GRiDD.github.io/main/gnet-sample-webhook.json](https://raw.githubusercontent.com/GRiDD/GRiDD.github.io/main/gnet-sample-webhook.json)

## Testing

### GBOOK
To test your incoming trips, you can log in to the gBook utility as:

User: (use username/password provided by GNET)

gBook URL: [https://gnet.grdd.net/gbook](https://gnet.grdd.net/gbook)

1. Login
2. Go to the BOOK menu
3. Select `<griddid>` from the affiliate dropdown box.
4. Fill in the trip details and click BOOK.
- When you click BOOK, you will receive the trip in your Waynium system.
- If you click SHOW JSON, it will show you the trip payload you need to submit to GNet for sending this trip.

### Note:
All the data you see on this screen should come to you in your trips. If anything is missing, then we have to fix it to make sure you get everything.

### Pricing:
We checked the pricing feature and that part is not implemented yet. We need information from you on where in your API we can find pricing and how it should be processed. Once we have that working, then the QUOTE feature should work for you as well.

If you could send some info on the pricing, we can work in parallel and put that up for testing this week too.

### POSTMAN
- Download Collection: [https://assets.grdd.dev/assets/GNET.postman_collection.json](https://assets.grdd.dev/assets/GNET.postman_collection.json)
- Get Token:
  - Provide `uid: user_id` and `pw: password`
  - Get token from the response and use it for the next step

- Set Variables:
  - `token`
  - `requesterId`
  - `providerId`

- Send Trip (Farm out)
  - Check the JSON payload and adjust values if needed

## Utilities

### Health Check
We will be using FarmIn Adapter GET for health check.

#### Expected Result:
Status code 200
```json
{
  "success": true
}
```
