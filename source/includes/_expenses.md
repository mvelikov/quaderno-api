# Expenses

Expenses are all the invoices that you receive from your vendors.

## Create an expense

> `POST /expenses.json`

```shell
# body.json

{     
  "contact_id":"5059bdbf2f412e0901000024",
  "contact_name":"ACME",
  "currency":"USD",
  "items_attributes":[
    {
    "description":"Rocket launcher",
    "quantity":"1.0",
    "unit_price":"0.0",
    "discount_rate":"0.0",
    "reference":"item_code_X"
    }
  ],
}

# curl command
curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X POST \
     --data-binary @body.json \     'https://ACCOUNT_NAME.quadernoapp.com/api/v1/expenses.json'
```

```php?start_inline=1
$expense = new QuadernoExpense(array(
                                 'po_number' => '',
                                 'currency' => 'USD',
                                 'tag_list' => 'playboy, businessman'));
$item = new QuadernoDocumentItem(array(
                               'description' => 'Rocket launcher',
                               'unit_price' => 0.0,
                               'discount_rate' => 0.0,
                               'reference' => "ITEM_ID",
                               'quantity' => 1.0));
$contact = QuadernoContact::find('5059bdbf2f412e0901000024');

$expense->addItem($item);
$expense->addContact($contact);

$expense->save(); // Returns true (success) or false (error)
```

```ruby
params = {     
  contact_id: '5059bdbf2f412e0901000024',
  contact_name: 'ACME',
  currency: 'USD',
  items_attributes: [
    {
    description: 'Rocket launcher',
    quantity: '1.0',
    unit_price: '0.0',
    discount_rate: '0.0',
    reference: 'ITEM_ID'
    }
  ],
}
Quaderno::Expense.create(params) #=> Quaderno::Expense
```

```swift?start_inline=1
TODO!
```

`POST`ing to `/expenses.json` will create a new contact from the parameters passed.

This will return `201 Created` and the current JSON representation of the expense if the creation was a success, along with the location of the new expense in the `url` field.

### Mandatory Fields

Key          | Description
-------------|------------------------------------------------------------------------------------------
`contact_id` / `contact`       | Either the ID of an existing contact or the JSON object for an existing/new contact.
`items_attributes` | An array of hashes which contains the `description`, `quantity`, `unit_price` and `discount_rate` of each item. If you want to have stock tracking, also pass the item code as the `reference` attribute. You may also add items to an expense by passing the `reference` of a pre-existing item.

<aside class="notice">
If you pass a `contact` JSON object instead of a `contact_id`, and the first and last name combination does not match any of your existing contacts, a new one will be created, otherwise a new expense will be created for the existing contact.<br /><br />

<p>Please note that you can pass <strong>either</strong> contact_id or contact, but if you pass both then results may not be what you expect.</p>

<p>Allowed contact fields are the same as when creating a contact via the dedicated <a href="#contacts">Contacts API</a>.</p>
</aside>

### Expense States

Possible expense states are:

- `outstanding`
- `paid`
- `late`
- `archived`

You can set the expense state by passing the `state` attribute. Bear the following in mind:

- The `paid` state is only reachable by adding a `payment` to the expense and is also final, so **cannot be overwritten unless the payment is removed from the expense**.

### Create an attachment during expense creation

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

Valid file extensions are `pdf`, `txt`, `jpeg`, `jpg`, `png`, `xml`, `xls`, `doc`, `rtf` and `html`. Any other format will stop expense creation and return a `422 Unprocessable Entity` error.

## Retrieve: Get and filter all expenses

> `GET /expenses.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/v1/expenses.json'
```

```ruby
Quaderno::Expense.all() #=> Array
```

```php?start_inline=1
$expenses = QuadernoExpense::find(); // Returns an array of QuadernoExpense
```

```swift
let client = Quaderno.Client(/* ... */)

let readExpense = Expense.read()
client.request(readExpense) { response in
  // response will contain the result of the request.
}
```

```json
[
  {
    "id":"5076a6b92f412e0e2e00006c",
    "issue_date":"2012-10-11",
    "contact":{
      "id":"5059bdbf2f412e0901000024",
      "full_name":"ACME"
    },
    "street_line_1":"Verizon Center",
    "street_line_2":"",
    "city":"Washington DC",
    "region":"Washington DC",
    "postal_code":"",
    "po_number":"",
    "currency":"USD",
    "exchange_rate":"0.680309",
    "items":[
      {
        "id":"48151623423",
        "description":"Rockets",
        "quantity":"50.0",
        "unit_price":"125.0",
        "discount_rate":"0.0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "subtotal":"$6,250.00",
        "discount":"$0.00",
        "gross_amount":"$6,250.00"
      }
    ],
    "subtotal":"$6,250.00",
    "discount":"$0.00",
    "taxes":[],
    "total":"$6,250.00",
    "payments":[],
    "tag_list":["rockets", "acme"],
    "notes":"",
    "state":"outstanding",
    "url":"https://my-account.quadernoapp.com/api/v1/expenses/5076a6b92f412e0e2e00006c"
  },
  {
    "id":"5076a6b92f412e0e2e00016d",
    "issue_date":"2012-10-11",
    "contact":{
    "id":"5059bdbf2f412e0901000024",
    "full_name":"ACME"
    },
    "street_line_1":"Verizon Center",
    "street_line_2":"",
    "city":"Washington DC",
    "region":"Washington DC",
    "postal_code":"",
    "po_number":"",
    "currency":"USD",
    "exchange_rate":"0.680309",
    "items":[
      {
      "id":" 48151623424",
      "description":"TNT",
      "quantity":"100.0",
      "unit_price":"25.0",
      "discount_rate":"0.0",
      "tax_1_name":"",
      "tax_1_rate":"",
      "tax_2_name":"",
      "tax_2_rate":"",
      "subtotal":"$2,500.00",
      "discount":"$0.00",
      "gross_amount":"$2,500.00"
    }
    ],
    "subtotal":"$2,500.00",
    "discount":"$0.00",
    "taxes":[],
    "total":"$2,500.00",
    "payments":[],
    "tag_list":["tnt", "acme"],
    "notes":"",
    "state":"outstanding",
    "url":"https://my-account.quadernoapp.com/api/v1/expenses/5076a6b92f412e0e2e00016d"
  },
]
```

`GET`ting from `/expenses.json` will return all the user's expenses.

You can filter the results in a few ways:

- By `contact_name` or `po_number` by passing the `q` parameter in the url like `?q=KEYWORD`.
- By date range, passing the `date` parameter in the url like `?date=DATE1,DATE2`.
- By state, passing the `state` parameter like `?state=STATE`.
- By specific vendor, passing the vendor ID in the `contact` parameter, like `?contact=3231`.
- By `exchange_rate`, if the expense currency differs from your account currency.

## Retrieve: Get a single expense

> `GET /expenses/EXPENSE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/v1/expenses/EXPENSE_ID.json'
```

```ruby
Quaderno::Expense.find(EXPENSE_ID) #=> Quaderno::Expense
```

```php?start_inline=1
$expense = QuadernoExpense::find('EXPENSE_ID'); // Returns a QuadernoExpense
```

```swift
let client = Quaderno.Client(/* ... */)

let readExpense = Expense.read()
client.request(readExpense) { response in
  // response will contain the result of the request.
}
```

```json
{
  "id":"5076a6b92f412e0e2e00006c",
  "issue_date":"2012-10-11",
  "contact":{
    "id":"5059bdbf2f412e0901000024",
    "full_name":"ACME"
  },
  "street_line_1":"Verizon Center",
  "street_line_2":"",
  "city":"Washington DC",
  "region":"Washington DC",
  "postal_code":"",
  "po_number":"",
  "currency":"USD",
  "items":[
    {
      "id":"48151623423",
      "description":"Rockets",
      "quantity":"50.0",
      "unit_price":"125.0",
      "discount_rate":"0.0",
      "tax_1_name":"",
      "tax_1_rate":"",
      "tax_2_name":"",
      "tax_2_rate":"",
      "subtotal":"$6,250.00",
      "discount":"$0.00",
      "gross_amount":"$6,250.00"
    }
  ],
  "subtotal":"$6,250.00",
  "discount":"$0.00",
  "taxes":[],
  "total":"$6,250.00",
  "payments":[],
  "tag_list":["rockets", "acme"],
  "notes":"",
  "state":"outstanding",
  "url":"https://my-account.quadernoapp.com/api/v1/expenses/5076a6b92f412e0e2e00006c"
}
```

`GET`ting from `/expenses/EXPENSE_ID.json` will return that specific expense.

## Update an expense

> `PUT /expenses/EXPENSE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X PUT \
     -d '{ "payment_details":"Money in da bank" }'  \
     'https://ACCOUNT_NAME.quadernoapp.com/api/v1/expenses/EXPENSE_ID.json'
```

```ruby
params = {
    payment_details: 'Money in da bank'
}
Quaderno::Expense.update(EXPENSE_ID, params) #=> Quaderno::Expense
```

```php?start_inline=1
$expense->payment_details = 'Money in da bank';
$expense->save();
```

```swift?start_inline=1
// TODO
```

`PUT`ting to `/expenses/EXPENSE_ID.json` will update the expense with the passed parameters.

This will return `200 OK` along with the current JSON representation of the expense if successful.

### Save an invoice as an expense

```json
{
  "permalink":"c2d480801d0e577c6c3177ce0795c4bdaa480e22"
}
```

`GET`ting `/expenses/save.json` will save another account invoice as an expense using the permalink passed as a parameter.

This will return `201 Created`, with the current JSON representation of the expense if the creation was a success, along the location of the new expense in the `url` field.

If the user does not have access to create new expenses, you'll see `401 Unauthorized` or a `422` if the permalink is invalid or the invoice is not suitable to be saved.

## Delete an expense

> `DELETE /expenses/EXPENSE_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X DELETE 'https://ACCOUNT_NAME.quadernoapp.com/api/v1/expenses/EXPENSE_ID.json'
```

```ruby
Quaderno::Expense.delete(EXPENSE_ID) #=> Boolean
```

```php?start_inline=1
$expense->delete();
```

```swift?start_inline=1
// TODO
```

`DELETE`ing to `/expense/EXPENSE_ID.json` will delete the specified contact and returns `204 No Content` if successful.