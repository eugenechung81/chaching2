# Introduction

Cha-ching was original conceptualized at a Mastercard Hackathon for addressing simplified payments through a keyboard shortcut across all messaging platforms (iMessage, Facebook Message, What's app, Email).  Similiar to Slash, the user will access the keyboard by clicking on the iOS globe button and then inputting the dollar amount to send to the receiver through the chat window.  The sent link is a serialized tag tied to the transfer which the receiptant opens.  This link is a "deep-link" that can open to several options: either a link to install the app or a link to quickly accept the payment through a web portal.  The value proposition of this app is user can pay quickly through any messaging platform and with any payment gateway (Paypal, Venmo, regular ACH).

![<Diagram Here>](https://bitbucket.org/repo/4qj7Rk/images/932455350-dia1.png)

# Messaging Platform

Cha-ching has the capacity to send over any platform because the infrastructure lives inside the keyboard and is not a separate app.  This allows the user to use the keyboard within iMessage, What's app, Gchat, WeChat and not have install a seperate application each time.  The keyboard also allows users to send a payment without having to leave the context of the conversation, open another app, login and punch in the numbers to make the payment.  The advantage is the ability to make a payment within the keyboard itself.  The serialized tag that is sent is picked up from the recipient which will accept the payment if the recipient has the app installed or allow them to accept the payment through a web portal.

![dia2.png](https://bitbucket.org/repo/4qj7Rk/images/3862947278-dia2.png)

# Payment Gateways

Venmo, Paypal and Banks are all supported as methods of transfer in Cha-ching.  Users can connect through their preferred payment solution and make payments to the receipent with a different payment login.  This is done through the hawala payment eco-structure.  Cha-ching acts as a broker and holding account for each respective payment solution.  User A using Venmo will "SEND" to Cha-ching Venmo account.  User B is using Paypal will "ACCEPT" from Cha-ching Paypal account.  To the User A and User B, the logical flow is straight forward.  In terms of where the physical money resides, they lie in each respective holding account credited and debited each account.

![dia3.png](https://bitbucket.org/repo/4qj7Rk/images/3679561204-dia3.png)

# Common Flow - Rest APIs

This is the flow when external services like iOS will invoke in order to create users and transfers.

## Send
```python
# Create User
response = requests.post(
    url=BASE_URL + "/user",
    data=utils.to_json("""
    {
      "py/object": "datas.User",
      "user_id": "917-555-6666",
      "venmo_details": {
        "py/object": "datas.VenmoDetails",
        "access_code": "29665a58391e6a08f027a298c4e2f6bacf8614f9c82e5aa210fa0f838ad2301e",
        "venmo_id": "test-user-5",
        "auth_code": ""
      },
      "bank_details": null,
      "transfer_method": "VENMO",
      "paypal_details": null,
      "password": "test",
      "id": ""
    }
    """),
    headers={"Content-Type": "application/json"}
)
print response.content

# Login User
response = requests.post(
    BASE_URL + '/token',
    data={
        "user_id": "917-555-6666",
        "password": "test"
    })
print utils.from_json(response.content).get("data").get("token")
token = utils.from_json(response.content).get("data").get("token")

# Transfer (Create tag id)
rsp = requests.get(
    BASE_URL + '/transfer/send',
    auth=HTTPBasicAuth(token, 'x'),
    data={
        "amount": "2.50"
    }
)
print rsp.content
"""
{
  "data": {
    "paypay_approval": null,
    "tag_id": "7MP2JE",
    "tag_url": "http://localhost:5000/accept?tag_id=7MP2JE"
  },
  "status": true,
  "status_msg": ""
}
"""
```

## Receive
```python
# Create User
response = requests.post(
    url=BASE_URL + "/user",
    data=utils.to_json("""
    {
      "py/object": "datas.User",
      "user_id": "917-555-7777",
      "venmo_details": {
        "py/object": "datas.VenmoDetails",
        "access_code": "29665a58391e6a08f027a298c4e2f6bacf8614f9c82e5aa210fa0f838ad2301e",
        "venmo_id": "test-user-5",
        "auth_code": ""
      },
      "bank_details": null,
      "transfer_method": "VENMO",
      "paypal_details": null,
      "password": "test",
      "id": ""
    }
    """),
    headers={"Content-Type": "application/json"}
)
print response.content

# Login User
response = requests.post(
    BASE_URL + '/token',
    data={
        "user_id": "917-555-7777",
        "password": "test"
    })
print utils.from_json(response.content).get("data").get("token")
token = utils.from_json(response.content).get("data").get("token")

# Transfer (Receive)
rsp = requests.get(
    BASE_URL + '/transfer/receive',
    auth=HTTPBasicAuth(token, 'x'),
    data={
        "tag_id": "1234FF",
    }
)
print rsp.content
```

# Common Flow - Web

## Send
```
http://localhost:5000/send
```

![dia4.png](https://bitbucket.org/repo/4qj7Rk/images/1928730286-dia4.png)

## Receive
```
http://localhost:5000/accept?tag_id=1234FF
```

![dia5.png](https://bitbucket.org/repo/4qj7Rk/images/3124934842-dia5.png)

# Verification

```python
# Verify
# http://localhost:5000/users
# http://localhost:5000/transfers
"""
{
  "bank_details": null,
  "id": "a909f9c6-16a8-40ab-9481-7fd2f79a1c7f",
  "password": "",
  "password_hash": "$5$rounds=571829$F7wsGAlflbgRqyjh$9voqvHDYJs7TltCqvXOMyU8rbgMxNaXjx06XcCGFJPD",
  "paypal_details": null,
  "transfer_method": "VENMO",
  "user_id": "917-555-6666",
  "venmo_details": {
    "access_code": "29665a58391e6a08f027a298c4e2f6bacf8614f9c82e5aa210fa0f838ad2301e",
    "auth_code": "",
    "venmo_id": "test-user-5"
  }
}

{
  "amount": 2.5,
  "currency": "USD",
  "description": "[Cha-Ching] 917-555-6666 -> None : $2.50",
  "destination_details": null,
  "destination_method": "",
  "destination_user_id": null,
  "dwolla_xfer_a2m_uuid": "",
  "dwolla_xfer_m2b_uuid": "",
  "id": "9215dcbb-9cbd-47cd-b4a0-12690e194374",
  "initiated_date": null,
  "logs": [
    "[2015-12-15 01:04:05] A->Mid Transaction Complete ([User user_id=917-555-6666, method=VENMO, details=(None, [Venmo access_code=29665a58391e6a08f027a298c4e2f6bacf8614f9c82e5aa210fa0f838ad2301e], None)])",
    "[2015-12-15 01:04:05] Tag Id Generated (7MP2JE)"
  ],
  "source_details": null,
  "source_method": "",
  "source_user_id": "917-555-6666",
  "status": "",
  "tag_id": "7MP2JE"
},
"""
```
