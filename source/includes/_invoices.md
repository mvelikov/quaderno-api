# Invoices

An invoice is a detailed list of goods shipped or services rendered, with an account of all costs.

## Create an invoice

> `POST /invoices.json`

```shell
# body.json
{
  "contact_id":"5059bdbf2f412e0901000024",
  "contact_name":"STARK",
  "po_number":"",
  "currency":"USD",
  "tag_list":"playboy, businessman",
  "items_attributes":[
    {
      "description":"Whiskey",
      "quantity":"1.0",
      "unit_price":"20.0",
      "discount_rate":"0.0",
      "reference":"item_code_X"
    }
  ]
}

curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X POST \
     --data-binary @body.json \
     'https://ACCOUNT_NAME.quadernoapp.com/api/invoices.json'
```

```php?start_inline=1
$invoice = new QuadernoInvoice(array(
                                 'po_number' => '',
                                 'currency' => 'USD',
                                 'tag_list' => array('playboy', 'businessman')));
$item = new QuadernoDocumentItem(array(
                               'description' => 'Pizza bagels',
                               'unit_price' => 9.99,
                               'quantity' => 20));
$contact = QuadernoContact::find('5059bdbf2f412e0901000024');

$invoice->addItem($item);
$invoice->addContact($contact);

$invoice->save(); // Returns true (success) or false (error)
```

```ruby
contact = Quaderno::Contact.find('50603e722f412e0435000024') #=> Quaderno::Contact

params = {
 contact_id: contact.id,
 po_number: 'PO.12234',
 currency: 'USD',
 tag_list: ['playboy', 'businessman'],
 items_attributes: [
  {
    description: 'Whiskey',
    quantity: '1.0',
    unit_price: '20.0',
    discount_rate: '0.0'
  }
 ]
}
invoice = Quaderno::Invoice.create(params) #=> Quaderno::Invoice

```

```swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let params : [String: Any] = [
 "contact_id": "5059bdbf2f412e0901000024",
 "contact_name": "STARK",
 "po_number": "",
 "currency": "USD",
 "tag_list": ["playboy", "businessman"]
]

let createInvoice = Invoice.create(params)
client.request(createInvoice) { response in
    // response will contain the result of the request.
}
```

`POST`ing to `/invoices.json` will create a new invoice from the parameters passed.

This will return `201 Created` and the current JSON representation of the invoice if the creation was a success, along with the location of the new invoice in the `url` field.

### Attributes

Attribute       | Mandatory                                  | Type/Description
----------------|--------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
number          | no                                         | String(255 chars) Available for invoices, credit_notes, receipts and estimates. Automatic numbering is used by default. Validates uniqueness
issue_date      | no                                         | String(255 chars) Available for all except for `recurring`. Defaults to the current date. Format YYYY-MM-DD
po_number       | no                                         | String(255 chars)
due_date        | no                                         | String(255 chars) Available for invoices, expenses and credit notes. Format `YYYY-MM-DD`
currency        | no                                         | String(3 chars) Defaults to the account currency. En formato ISO 4217
tag_list        | no                                         | String(255 chars). Multiple tags should be separated by commas
payment_details | no                                         | Text
notes           | no                                         | Text
contact_id      | (Mandatory if `contact` is not present)    | ID
contact         | (Mandatory if `contact_id` is not present) | Hash with a contact data for creation
street_line_1   | no                                         | String(255 chars). Available for updates
street_line_2   | no                                         | String(255 chars)
city            | no                                         | String(255 chars). Available for updates
region          | no                                         | String(255 chars). Available for updates
postal_code     | no                                         | String(255 chars). Available for updates
items_attributes| **yes**                                        | Array of document items (check available attributes for document items below)
payment_method  | no                                         | Create a paid document in a single request. One of the following: `credit_card`, `cash`, `wire_transfer`, `direct_debit`, `check`, `promissory_note`, `iou`, `paypal` or `other`

<aside class="notice">
If you pass a `contact` JSON object instead of a `contact_id`, and the first and last name combination does not match any of your existing contacts, a new one will be created, otherwise a new invoice will be created for the existing contact. Only a `contact` object OR a `contact_id` property should be passed in the same call. Only a `contact` object OR a `contact_id` property should be passed in the same call.<br /><br />

<p>Please note that you can pass <strong>either</strong> contact_id or contact, but if you pass both then results may not be what you expect.</p>

<p>Allowed contact fields are the same as when creating a contact via the dedicated <a href="#contacts">Contacts API</a>.</p>
</aside>

### Document Item Attributes

Attribute     | Mandatory                                | Type/Description
--------------|------------------------------------------|----------------------------------------------------------------------------------------------------------------------
id            | no                                       | ID. Available only for updates
description   | **yes**                                  | String(255 chars)
quantity      | no                                       | Decimal. Defaults to 1.0
unit_price    | **yes**                                  | Decimal
total_amount  | Mandatory if `unit_price` is not present | Decimal
discount_rate | no                                       | Decimal
tax_1_name    | Mandatory if `tax_1_rate` is present     | String(255 chars)
tax_1_rate    | Mandatory if `tax_1_name` is present     | Decimal between -100.00 and 100.00 (not included)
tax_1_country | no                                       | String(2 chars). Defaults to the contact's country
tax_2_name    | Mandatory if `tax_2_rate` is present     | String(255 chars)
tax_2_rate    | Mandatory if `tax_2_name` is present     | Decimal between -100.00 and 100.00 (not included)
tax_2_country | no                                       | String(2 chars). Defaults to the contact's country
reference     | no                                       | String(255 chars) Code (`code`) of an existing item. **If present none of the mandatory attributes are mandatory (as you are referencing an item that already exists)**
_destroy      | no                                       | Set it to 1 if you want to remove the document item selected by ID. Available only for updates

### Invoice States

Possible invoice states are:

- `draft`
- `sent`
- `partial`
- `paid`
- `late`
- `archived`

### Create an attachment during invoice creation

```json
{
  "attachment":{
    "data":"aBaSe64EnCoDeDFiLe",
    "filename":"the_filename.png"
  }
}
```

Optionally, you can pass an attachment hash along with the rest of the parameters.

Fields:

Field      | Description
-----------|------------------------------------------------------------
`data`     | Contains a Base64 encoded string which represents the file.
`filename` | The attachment file name.

Valid file extensions are `pdf`, `txt`, `jpeg`, `jpg`, `png`, `xml`, `xls`, `doc`, `rtf` and `html`. Any other format will stop invoice creation and return a `422 Unprocessable Entity` error.

## Retrieve: Get and filter all invoices

> `GET /invoices.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/invoices.json'
```

```ruby
Quaderno::Invoice.all() #=> Array
```

```php?start_inline=1
$invoices = QuadernoInvoice::find(); // Returns an array of QuadernoInvoice
```

```swift
let client = Quaderno.Client(/* ... */)

let listInvoices = Invoice.list(pageNum)
client.request(listInvoices) { response in
  // response will contain the result of the request.
}
```

```json
[
  {
    "id":"507693322f412e0e2e00000f",
    "number":"0000047",
    "issue_date":"2012-10-11",
    "contact":{
      "id":"5073f9c22f412e02d0000032",
      "full_name":"Garfield"
    },
    "street_line_1":"Street 23",
    "street_line_2":"",
    "city":"New York",
    "region":"New York",
    "postal_code":"10203",
    "po_number":null,
    "due_date":null,
    "currency":"USD",
    "exchange_rate":"0.680309",
    "items":[
      {
        "id":"48151623429",
        "description":"lasagna",
        "quantity":"25.0",
        "unit_price_cents":"375",
        "discount_rate":"0.0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "reference":"Awesome!",
        "subtotal_cents":"9375",
        "discount_cents":"0",
        "gross_amount_cents":"9375"
      }
    ],
    "subtotal_cents":"9375",
    "discount_cents":"0",
    "taxes":[],
    "total_cents":"9375",
    "payments":[
      {
        "id":"50aca7d92f412eda5200002c",
        "date":"2012-11-21",
        "payment_method":"credit_card",
        "amount_cents":"9375",
        "url":"https://ACCOUNT_NAME.quadernoapp.com/api/invoices/507693322f412e0e2e00000f/payments/50aca7d92f412eda5200002c.json"
      },
    ],
    "payment_details":"Ask Jon",
    "notes":"",
    "state":"paid",
    "tag_list":["lasagna", "cat"],
    "secure_id":"7hef1rs7p3rm4l1nk",
    "permalink":"https://quadernoapp.com/invoice/7hef1rs7p3rm4l1nk",
    "pdf":"https://quadernoapp.com/invoice/7hef1rs7p3rm4l1nk.pdf",
    "url":"https://ACCOUNT_NAME.quadernoapp.com/api/invoices/507693322f412e0e2e00000f.json"
  },

  {
    "id":"507693322f412e0e2e0000da",
    "number":"0000047",
    "issue_date":"2012-10-11",
    "contact":{
      "id":"5073f9c22f412e02d00004cf",
      "full_name":"Teenage Mutant Ninja Turtles"
    },
    "street_line_1":"Melrose Ave, Sewer #3",
    "street_line_2":"",
    "city":"New York",
    "region":"New York",
    "postal_code":"",
    "po_number":null,
    "due_date":null,
    "currency":"USD",
    "exchange_rate":"0.680309",
    "items":[
      {
        "id":"481516234291",
        "description":"pizza",
        "quantity":"15.0",
        "unit_price_cents":"6000",
        "discount_rate":"0.0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "reference":"Even the bad ones taste good!",
        "subtotal_cents":"6000",
        "discount_cents":"0",
        "gross_amount_cents":"6000"
      }
    ],
    "subtotal_cents":"6000",
    "discount_cents":"0",
    "taxes":[],
    "total_cents":"6000",
    "payments":[],
    "payment_details":"",
    "notes":"",
    "state":"draft",
    "tag_list":["pizza", "turtles"],
    "secure_id":"7hes3c0ndp3rm4l1nk",
    "permalink":"https://ACCOUNT_NAME.quadernoapp.com/invoice/7hes3c0ndp3rm4l1nk",
    "pdf":"https://ACCOUNT_NAME.quadernoapp.com/invoice/7hes3c0ndp3rm4l1nk.pdf",
    "url":"https://ACCOUNT_NAME.quadernoapp.com/api/invoices/507693322f412e0e2e0000da.json"
  },
]
```

`GET`ting from `/invoices.json` will return all the user's invoices.

You can filter the results in a few ways:

- By `number`, `contact_name` or `po_number` by passing the `q` parameter in the url like `?q=KEYWORD`.
- By date range, passing the `date` parameter in the url like `?date=DATE1,DATE2`.
- By state, passing the `state` parameter like `?state=STATE`.
- By contact, passing the contact ID in the `contact` parameter, like `?contact=3231`.

## Retrieve: Get a single invoice

> `GET /invoices/INVOICE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/invoices/INVOICE_ID.json'
```

```ruby
Quaderno::Invoice.find(INVOICE_ID) #=> Quaderno::Invoice
```

```php?start_inline=1
$invoice = QuadernoInvoice::find('INVOICE_ID'); // Returns a QuadernoInvoice
```

```swift
let client = Quaderno.Client(/* ... */)

let readInvoice = Invoice.read(INVOICE_ID)
client.request(readInvoice) { response in
  // response will contain the result of the request.
}
```

```json
{
  "id":"507693322f412e0e2e0000da",
  "number":"0000047",
  "issue_date":"2012-10-11",
  "contact":{
    "id":"5073f9c22f412e02d00004cf",
    "full_name":"Teenage Mutant Ninja Turtles"
  },
  "street_line_1":"Melrose Ave, Sewer #3",
  "street_line_2":"",
  "city":"New York",
  "region":"New York",
  "postal_code":"",
  "po_number":null,
  "due_date":null,
  "currency":"USD",
  "exchange_rate":"0.680309",
  "items":[
    {
      "id":"48151623429",
      "description":"pizza",
      "quantity":"15.0",
      "unit_price_cents":"6000",
      "discount_rate":"0.0",
      "tax_1_name":"",
      "tax_1_rate":"",
      "tax_2_name":"",
      "tax_2_rate":"",
      "reference":"Even the bad ones taste good!",
      "subtotal_cents":"6000",
      "discount_cents":"0",
      "gross_amount_cents":"6000"
    }
  ],
  "subtotal_cents":"6000",
  "discount_cents":"0",
  "taxes":[],
  "total_cents":"6000",
  "payments":[],
  "payment_details":"",
  "notes":"",
  "state":"draft",
  "tag_list":["lasagna", "cat"],
  "secure_id":"7hef1rs7p3rm4l1nk",
  "permalink":"https://ACCOUNT_NAME.quadernoapp.com/invoice/7hef1rs7p3rm4l1nk",
  "pdf":"https://ACCOUNT_NAME.quadernoapp.com/invoice/7hef1rs7p3rm4l1nk.pdf",
  "url":"https://ACCOUNT_NAME.quadernoapp.com/api/invoices/507693322f412e0e2e0000da.json"
}
```

`GET`ting from `/invoices/INVOICE_ID.json` will return that specific invoice.

If you have connected Quaderno and Stripe, you can also `GET /stripe/charges/STRIPE_CHARGE_ID.json` to get the Quaderno invoice for a Stripe charge.

## Update an invoice

> `PUT /invoices/INVOICE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X PUT \
     -d '{"notes":"You better pay this time, Tony."}' \
     'https://ACCOUNT_NAME.quadernoapp.com/api/invoices/INVOICE_ID.json'
```

```ruby
params = {
    notes: 'You better pay this time, Tony.'
}
Quaderno::Invoice.update(INVOICE_ID, params) #=> Quaderno::Invoice
```

```php?start_inline=1
$invoice->notes = 'You better pay this time, Tony.';
$invoice->save();
```

````swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let params : [String: Any] = [
    "notes": "You better pay this time, Tony."
]

let updateInvoice = Invoice.update(INVOICE_ID, params)
client.request(updateInvoice) { response in
    // response will contain the result of the request.
}
```

`PUT`ting to `/invoices/INVOICE_ID.json` will update the invoice with the passed parameters.

This will return `200 OK` along with the current JSON representation of the invoice if successful.

## Delete an invoice

> `DELETE /invoices/INVOICE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X DELETE 'https://ACCOUNT_NAME.quadernoapp.com/api/invoices/INVOICE_ID.json'
```

```ruby
Quaderno::Invoice.delete(INVOICE_ID) #=> Boolean
```

```php?start_inline=1
$invoice->delete();
```

```swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let deleteInvoice = Invoice.delete(INVOICE_ID)
client.request(deleteInvoice) { response in
    // response will contain the result of the request.
}
```

`DELETE`ing to `/invoices/INVOICE_ID.json` will delete the specified invoice and returns `204 No Content` if successful.

## Deliver (Send) an invoice

```shell
curl -u YOUR_API_KEY: \
     -X GET \
     'https://ACCOUNT_NAME.quadernoapp.com/api/invoices/INVOICE_ID/deliver.json'
```

```ruby
invoice = Quaderno::Invoice.find(INVOICE_ID)
invoice.deliver
```

```php?start_inline=1
$invoice->deliver(); // Return true (success) or false (error)
```

```swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let deliverInvoice = Invoice.deliver(INVOICE_ID)
client.request(deliverInvoice) { response in
  // response will contain the result of the request.
}
```

`GET`ting `/invoices/INVOICE_ID/deliver.json` will send the invoice to the assigned contact email. This will return `200 OK` if successful, along with a JSON representation of the invoice.

If the destination contact does not have an email address you will receive a `422 Unprocessable Entity` error.
