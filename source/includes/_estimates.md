# Estimates

An estimate is an offer that you give a client in order to get a specific job. With time, estimates are usually turned into issued invoices.

## Create an estimate

> `POST /estimates.json`

```shell
# body.json

{
  "number":"0000006",
  "contact_id":"50603e722f412e0435000024",
  "contact_name":"Wild E. Coyote",
  "po_number":"",
  "currency":"EUR",
  "items_attributes":[
    {
        "description":"ACME Catapult",
        "quantity":"1.0",
        "unit_price":"0.0",
        "discount_rate":"0.0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "reference":"item_code_X"
      }
  ],
  "tag_list":"tnt",
  "payment_details":"",
  "notes":""
}

# curl command
curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X POST \
     --data-binary @- body.json \
     'https://ACCOUNT_NAME.quadernoapp.com/api/v1/estimates.json'
```

```php?start_inline=1
$estimate = new QuadernoEstimate(array(
                                 'po_number' => '',
                                 'currency' => 'USD',
                                 'tag_list' => 'playboy, businessman'));
$item = new QuadernoDocumentItem(array(
                               'description' => 'ACME Catapult',
                               'unit_price' => 0.0,
                               'reference' => 'ITEM_ID',
                               'quantity' => 1.0));
$contact = QuadernoContact::find('5059bdbf2f412e0901000024');

$estimate->addItem($item);
$estimate->addContact($contact);

$estimate->save(); // Returns true (success) or false (error)
```

```ruby
params = {
  number: '0000006',
  contact_id: '50603e722f412e0435000024',
  contact_name: 'Wile E. Coyote',
  po_number: '',
  currency: 'EUR',
  items_attributes: [
    {
        description: 'ACME Catapult',
        quantity: '1.0',
        unit_price: '0.0',
        discount_rate: '0.0',
        tax_1_name: '',
        tax_1_rate: '',
        tax_2_name: '',
        tax_2_rate: '',
        reference: 'ITEM_ID'
      }
  ],
  tag_list: 'tnt',
  payment_details: '',
  notes: ''
}
Quaderno::Estimate.create(params) #=> Quaderno::Estimate
```

```swift?start_inline=1
TODO!
```

`POST`ing to `/estimates.json` will create a new estimate from the parameters passed.

This will return `201 Created` and the current JSON representation of the estimate if the creation was a success, along with the location of the new estimate in the `url` field.

### Mandatory Fields

Key          | Description
-------------|------------------------------------------------------------------------------------------
`contact_id` / `contact`       | Either the ID of an existing estimate or the JSON object for an existing/new estimate.
`items_attributes` | An array of hashes which contains the `description`, `quantity`, `unit_price` and `discount_rate` of each item. If you want to have stock tracking, also pass the item code as the `reference` attribute. You may also add items to an estimate by passing the `reference` of a pre-existing item.

<aside class="notice">
If you pass a `contact` JSON object instead of a `contact_id`, and the first and last name combination does not match any of your existing estimates, a new one will be created, otherwise a new estimate will be created for the existing estimate.<br /><br />

<p>Please note that you can pass <strong>either</strong> estimate_id or estimate, but if you pass both then results may not be what you expect.</p>

<p>Allowed estimate fields are the same as when creating a estimate via the dedicated <a href="#estimates">Contacts API</a>.</p>
</aside>

### Estimate States

Possible estimate states are:

- `draft`
- `sent`
- `accepted`
- `declined`
- `invoiced`
- `reverted`
- `late`

You can set the estimate state by passing the `state` attribute, but bear the following considerations in mind:

- The `invoiced` state is final and cannot be overwritten.
- The `sent` state can only overwrite the state `draft`.

### Create an attachment during estimate creation

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

Valid file extensions are `pdf`, `txt`, `jpeg`, `jpg`, `png`, `xml`, `xls`, `doc`, `rtf` and `html`. Any other format will stop estimate creation and return a `422 Unprocessable Entity` error.

## Retrieve: Get and filter all estimates

> `GET /estimates.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/v1/estimates.json'
```

```ruby
Quaderno::Estimate.all() #=> Array
```

```php?start_inline=1
$estimates = QuadernoEstimate::find(); // Returns an array of QuadernoEstimate
```

```swift
let client = Quaderno.Client(/* ... */)

let readEstimate = Estimate.read()
client.request(readEstimate) { response in
  // response will contain the result of the request.
}
```

```json
[
  {  
    "id":"50603e722f412e0435000024",
    "number":"0000003",
    "issue_date":"2012-09-24",
    "contact":{
      "id":"5059bdbf2f412e0901000024",
      "full_name":"Wild E. Coyote"
    },
    "street_line_1":"Desert of New Mexico",
    "street_line_2":"",
    "city":"New Mexico",
    "region":"New Mexico",
    "postal_code":"",
    "po_number":"",
    "currency":"EUR",
    "items":[
      {
        "id":"4815162342",
        "description":"ACME TNT",
        "quantity":"1.0",
        "unit_price":"100.0",
        "discount_rate":"0.0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "subtotal":"\u20ac100.00",
        "discount":"\u20ac0.00",
        "gross_amount":"\u20ac100.00"
       }
    ],
    "subtotal":"\u20ac100.00",
    "discount":"\u20ac0.00",
    "taxes":[],
    "total":"\u20ac100.00",
    "payment_details":"",
    "notes":"",
    "state":"draft",
    "tag_list":[],
    "permalink":"https://my-account.quadernoapp.com/estimate/7hef1rs7p3rm4l1nk",
    "url":"https://my-account.quadernoapp.com/api/v1/estimates/50603e722f412e0435000024.json"
  },
  {  
    "id":"50603e722f412e0435000144",
    "number":"0000005",
    "issue_date":"2012-09-24",
    "contact":{
      "id":"5059bdbf2f412e0901000044",
      "full_name":"Cookie Monster"
    },
    "street_line_1":"Sesame Street",
    "street_line_2":"",
    "city":"New York",
    "region":"New York",
    "postal_code":"",
    "po_number":"",
    "currency":"EUR",
    "items":[
      {
        "id":"48151623421",
        "description":"Cookies",
        "quantity":"5.0",
        "unit_price":"1.95",
        "discount_rate":"0.0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "subtotal":"\u20ac9.75",
        "discount":"\u20ac0.00",
        "gross_amount":"\u20ac9.75"
       }
    ],
    "subtotal":"\u20ac9.75",
    "discount":"\u20ac0.00",
    "taxes":[],
    "total":"\u20ac9.75",
    "payment_details":"",
    "notes":"",
    "state":"draft",
    "tag_list":[],
    "permalink":"https://my-account.quadernoapp.com/estimate/7hes3c0ndp3rm4l1nk",
    "url":"https://my-account.quadernoapp.com/api/v1/estimates/50603e722f412e0435000144.json"
  }
]
```

`GET`ting from `/estimates.json` will return all the user's estimates.

You can filter the results in a few ways:

- By `number`, `contact_name` or `po_number` by passing the `q` parameter in the url like `?q=KEYWORD`.
- By date range, passing the `date` parameter in the url like `?date=DATE1,DATE2`.
- By state, passing the `state` parameter like `?state=STATE`.
- By contact, passing the contact ID in the `contact` parameter, like `?contact=3231`.
- By `exchange_rate`, if the estimate currency differs from your account currency.

## Retrieve: Get a single estimate

> `GET /estimates/ESTIMATE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/v1/estimates/ESTIMATE_ID.json'
```

```ruby
Quaderno::Estimate.find(ESTIMATE_ID) #=> Quaderno::Estimate
```

```php?start_inline=1
$estimate = QuadernoEstimate::find('ESTIMATE_ID'); // Returns a QuadernoEstimate
```

```swift
let client = Quaderno.Client(/* ... */)

let readEstimate = Estimate.read()
client.request(readEstimate) { response in
  // response will contain the result of the request.
}
```

```json
{  
  "id":"50603e722f412e0435000024",
  "number":"0000003",
  "issue_date":"2012-09-24",
  "contact":{
    "id":"5059bdbf2f412e0901000024",
    "full_name":"Wild E. Coyote"
  },
  "street_line_1":"Desert of New Mexico",
  "street_line_2":"",
  "city":"New Mexico",
  "region":"New Mexico",
  "postal_code":"",
  "po_number":"",
  "currency":"EUR",
  "items":[
    {
      "id":"4815162342"
      "description":"ACME TNT",
      "quantity":"1.0",
      "unit_price":"\u20ac100.0",
      "discount_rate":"0.0",
      "tax_1_name":"",
      "tax_1_rate":"",
      "tax_2_name":"",
      "tax_2_rate":"",
      "subtotal":"\u20ac100.00",
      "discount":"\u20ac0.00",
      "gross_amount":"\u20ac100.00"
    }
  ],
  "subtotal":"\u20ac100.00",
  "discount":"\u20ac0.00",
  "taxes":[],
  "total":"\u20ac100.00",
  "payment_details":"",
  "notes":"",
  "state":"draft",
  "tag_list":[],
  "permalink":"https://my-account.quadernoapp.com/estimate/7hef1rs7p3rm4l1nk",
  "url":"https://my-account.quadernoapp.com/api/v1/estimates/50603e722f412e0435000024.json"
}
```

`GET`ting from `/estimates/ESTIMATE_ID.json` will return that specific estimate.

## Update an estimate

> `PUT /estimates/ESTIMATE_ID.json`

```shell
# body.json

{
  "tag_list":"Wacky, racer"
  "contact_id":"505c3b402f412e0248000044",
  "contact_name":"Dick Dastardly",
}

curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X PUT \
     --data-binary @body.json \
     'https://ACCOUNT_NAME.quadernoapp.com/api/v1/estimates/ESTIMATE_ID.json'
```

```ruby
params = {
    contact_id: '505c3b402f412e0248000044',
    contact_name: 'Dick Dastardly'
}
Quaderno::Estimate.update(ESTIMATE_ID, params) #=> Quaderno::Estimate
```

```php?start_inline=1
$estimate->contact_id = '505c3b402f412e0248000044';
$estimate->contact_name = 'Dick Dastardly';
$estimate->save();
```

```swift?start_inline=1
// TODO
```

`PUT`ting to `/estimates/ESTIMATE_ID.json` will update the estimate with the passed parameters.

This will return `200 OK` along with the current JSON representation of the estimate if successful.

## Delete an estimate

> `DELETE /estimates/ESTIMATE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X DELETE 'https://ACCOUNT_NAME.quadernoapp.com/api/v1/estimates/ESTIMATE_ID.json'
```

```ruby
Quaderno::Estimate.delete(ESTIMATE_ID) #=> Boolean
```

```php?start_inline=1
$estimate->delete();
```

```swift?start_inline=1
// TODO
```

`DELETE`ing to `/estimate/ESTIMATE_ID.json` will delete the specified estimate and returns `204 No Content` if successful.

## Deliver (Send) an estimate

`GET`ting `/estimates/ESTIMATE_ID/deliver.json` will send the invoice to the assigned contact email. This will return `200 OK` if successful, along with a JSON representation of the invoice.

If the destination contact does not have an email address you will receive a `422 Unprocessable Entity` error.