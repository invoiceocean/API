# InvoiceOcean API


Description on how to integrate own apllication or service with <http://invoiceocean.com/> system



Thanks to API you can issue invoices/bills/receipts from other systems and manage these documents, as well as clients and products

## Table of contents
+ [API Token](#token)  
+ [Invoices - examples of calling](#examples)  
	+ [Downloading a list of invoices from current month](#1)
	+ [Specific client's invoices](#2)
	+ [Downloading invoices by ID](#3)
	+ [Downloading as PDF](#4)
	+ [Sending invoices by email to a client](#5)
	+ [Adding a new invoice](#6)
	+ [Adding a new invoice (by client, product, seller ID)](#7)
	+ [Adding a new correction invoice](#8)
	+ [Invoice update](#9)
	+ [Changing invoice status](#10)
	+ [Downloading a definition list of recurring invoices](#11)
	+ [Adding a new definition of recurring invoice](#12)
	+ [Actualizing a definition of recurring invoice](#13)
	+ [Deleting an invoice](#14)
+ [Link to invoice preview and PDF download](#view_url)  
+ [Examples of use - purchase of training](#use_case1)  
+ [Invoices - specification](#invoices)
+ [Clients](#clients)
	+ [Clients list](#c1)
	+ [Get selected client by ID](#c2)
	+ [Adding clients](#c3)
	+ [Client update](#c4)
+ [Products](#products)
	+ [Products list](#p1)
	+ [Products list with storage quantities for a specific magazine](#p2)
	+ [Get selected product by ID](#p3)
	+ [Get selected product by ID with storage quantity for a specific magazine](#p4)
	+ [Adding products](#p5)
	+ [Product update](#p6)
+ [Payments](#payments)
+ [Examples in PHP and Ruby](#codes)  


<a name="token"/>
##API token

`API_TOKEN` token has to be downloaded from application settings ("Settings -> Account settings -> Integration -> API Authorization Code")

<a name="examples"/>
##Examples of calling

<a name="1"/>
Downloading a list of invoices from current month: 

```shell
curl https://yourdomain.invoiceocean.com/invoices.json?period=this_month&api_token=API_TOKEN
```

<b>NOTE</b>: additional parameters can be forwarded to calls, e.g. `page=`, `period=` etc.

<a name="2"/>
Specific client's invoices: 

```shell
curl https://yourdomain.invoiceocean.com/invoices.json?client_id=ID_KLIENTA&api_token=API_TOKEN
```

<a name="3"/>
Downloading invoices by ID: 

```shell
curl https://yourdomain.invoiceocean.com/invoices/100.json?api_token=API_TOKEN
```

<a name="4"/>
Downloading as PDF: 

```shell
curl https://yourdomain.invoiceocean.com/invoices/100.pdf?api_token=API_TOKEN
```

<a name="5"/>
Sending invoices by email to a client: 

```shell
curl -X POST https://yourdomain.invoiceocean.com/invoices/100/send_by_email.json?api_token=API_TOKEN
```

Other PDF options:
* print_option=original - Original
* print_option=copy - Copy
* print_option=original_and_copy - Original and copy
* print_option=duplicate Duplicate

<a name="6"/>
Adding a new invoice: 

```shell
curl https://YOUR_DOMAIN.invoiceocean.com/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "kind":"vat", 
            "number": null, 
            "sell_date": "2013-01-16", 
            "issue_date": "2013-01-16", 
            "payment_to": "2013-01-23",
            "seller_name": "Wystawca Sp. z o.o.", 
            "seller_tax_no": "5252445767", 
            "buyer_name": "Klient1 Sp. z o.o.",
            "buyer_tax_no": "5252445767",
            "positions":[
                {"name":"Produkt A1", "tax":23, "total_price_gross":10.23, "quantity":1},
                {"name":"Produkt A2", "tax":0, "total_price_gross":50, "quantity":3}
            ]       
        }
    }'
```

<a name="7"/>
Adding a new invoice - the minimal version (only fields required), when we have product, buyer and seller ID we do not need to provide full details.
VAT Invoice with current date and 5 day due date will be issued: 

```shell
curl http://YOUR_DOMAIN.invoiceocean.com/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "invoice": {
            "payment_to_kind": 5,
            "department_id": 1, 
            "client_id": 1,
            "positions":[
                {"product_id": 1, "quantity":2}
            ]
        }}'
```	   

<a name="8"/>
Adding a new correction invoice: 

```shell
curl http://YOUR_DOMAIN.invoiceocean.com/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "invoice": {
            "kind": "correction",
            "from_invoice_id": "2432393,
            "client_id": 1,
            "positions":[
                {"name": "Product A1",
                "quantity":-1,
                "total_price_gross":"-10",
                "tax":"23",
                "correction_before_attributes": {
                    "name":"Product A1",
                    "quantity":"2",
                    "total_price_gross":"20",
                    "tax":"23",
                    "kind":"correction_before"
                },
                "correction_after_attributes": {
                    "name":"Product A1",
                    "quantity":"1",
                    "total_price_gross":"10",
                    "tax":"23",
                    "kind":"correction_after"
                }
            }]
        }}'
```

<a name="9"/>
Invoice update: 

```shell
curl https://YOUR_DOMAIN.invoiceocean.com/invoices/111.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "invoice": {
            "buyer_name": "New client name Ltd."
        }
    }'
```

<a name="10"/>
Changing invoice status: 

```shell
curl "https://YOUR_DOMAIN.invoiceocean.com/invoices/111/change_status.json?api_token=API_TOKEN&status=STATUS" -X POST
```

<a name="11"/>
Downloading a definition list of recurring invoices: 

```shell
curl https://YOUR_DOMAIN.fakturownia.pl/recurrings.json?api_token=API_TOKEN
```

<a name="12"/>
Adding a new definition of recurring invoice: 

```shell
curl https://YOUR_DOMAIN.fakturownia.pl/recurrings.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "recurring": {
            "name": "Nazwa cyklicznosci",
            "invoice_id": 1,
            "start_date": "2016-01-01",
            "every": "1m",
            "issue_working_day_only": false,
            "send_email": true,
            "buyer_email": "mail1@mail.pl, mail2@mail.pl",
            "end_date": "null"
        }}'
```

<a name="13"/>
Actualizing a definition of recurring invoice: 

```shell
curl https://YOUR_DOMAIN.fakturownia.pl/recurrings/111.json \
    -X PUT \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{
        "api_token": "API_TOKEN",
        "recurring": {
            "next_invoice_date": "2016-02-01"
        }
    }'
```

<a name="14"/>
Deleting an invoice: 

```shell
curl -X DELETE "http://YOUR_DOMAIN.fakturownia.pl/invoices/INVOICE_ID.json?api_token=API_TOKEN"
```

<a name="view_url"/>
##Link to invoice preview and PDF download

After downloading invoice data, e.g. by:

```shell
curl https://YOUR_DOMAIN.invoiceocean.com/invoices/100.json?api_token=API_TOKEN
```

API gives us `token` field, on which basis we may receive invoice preview links 
Such links allow you to refer to the selected invoice without having to log in - you can, for instance, send these links to the customer, who will have access to invoices and PDF.

Links are in the form: 

preview: `http://yourdomain.invoiceocean.com/invoice/{{token}}` 
pdf: `http://yourdomain.invoiceocean.com/invoice/{{token}}.pdf`

E.g. for token equal: `HBO3Npx2OzSW79RQL7XV2` public PDF will be at `http://yourdomain.invoiceocean.com/invoice/HBO3Npx2OzSW79RQL7XV2.pdf`

<a name="use_case1"/>
##Examples of using PHP - purchase of training

`TODO` 

Flow Portal Example which generates a proforma invoice for the client, sends it to the client and after receiving payment, sends the training ticket to the client

* Client fills in details in the Portal
* The Portal calls API from invoiceocean.com and generates an invoice
* The Portal sends a Proforma PDF invoice to the Client along with a payment link
* Client makes a payment for the Proforma invoice (e.g. using PayPal)
* InvoiceOcean.com receives information that the payment has been made, generates VAT invoice and sends it to the client and calls Portal API
* After receiving information regarding payment (by API) Portal sends the training ticket to the Client


<a name="invoices"/>
##Invoices


* `GET /invoices/1.json` downloading invoice
* `POST /invoices.json` adding a new invoice
* `PUT /invoices/1.json` updating invoice
* `DELETE /invoices/1.json` deleting invoice


Example - adding a new invoice - the minimal version (only fields required), when we have product, buyer and seller ID we do not need to provide full details. Field department_id determines the company (or department) which issues the invoice (it can be obtained by clicking on the company in Settings> Data Company)

```shell
curl http://YOUR_DOMAIN.invoiceocean.com/invoices.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "invoice": {
            "payment_to_kind": 5,
            "department_id": 1, 
            "client_id": 1,
            "positions":[
                {"product_id": 1, "quantity":2}
            ]
        }}'
```
 
Invoice fields

```shell
"number" : "13/2012" - invoice number (if not entered, it will be automatically generated)
"kind" : "vat" - invoice kind (vat, proforma, bill, receipt, advance, correction, vat_mp, invoice_other, vat_margin, kp, kw, final, estimate)
"income" : "1" - income invoice (1) or cost invoice (0)
"issue_date" : "2013-01-16" - date of issue 
"place" : "Warszawa" - place of issue
"sell_date" : "2013-01-16" - date of sale (it can be date or month in the YYYY-MM format)
"category_id" : "" - category id
"department_id" : "1" - department id (in Settings > Company / department, click on company / department and department ID will be shown in the URL)
"seller_name" : "Radgost Sp. z o.o." - seller
"seller_tax_no" : "525-244-57-67" - seller tax id
"seller_bank_account" : "24 1140 1977 0000 5921 7200 1001" - seller bank account
"seller_bank" : "BRE Bank", 
"seller_post_code" : "02-548", 
"seller_city" : "Warsaw", 
"seller_street" : "21 Olesińska St.", 
"seller_country" : "", 
"seller_email" : "platnosci@radgost123.com", 
"seller_www" : "", 
"seller_fax" : "", 
"seller_phone" : "", 
"client_id" : "-1" - buyer id (if -1 then client will be created in the system)
"buyer_name" : "Client name - buyer
"buyer_tax_no" : "525-244-57-67", 
"disable_tax_no_validation" : "", 
"buyer_post_code" : "30-314", 
"buyer_city" : "Warsaw", 
"buyer_street" : "Nowa 44", 
"buyer_country" : "", 
"buyer_note" : "", 
"buyer_email" : "", 
"additional_info" : "0" - whether to display additional field in invoice position
"additional_info_desc" : "PKWiU" - name of the additional column in invoice positions
"show_discount" : "0" - whether show discount or not
"payment_type" : "transfer", 
"payment_to_kind" : due date. if it is "other_date", then you may define a specific date in "payment_to" field, if it is, for example, numer 5 then you have a 5 day payment period
"payment_to" : "2013-01-16", 
"status" : "issued", 
"paid" : "0,00", 
"oid" : "zamowienie10021", - order number (e.g. from external ordering system)
"warehouse_id" : "1090", 
"seller_person" : "Forename Surname", 
"buyer_first_name" : "Forename", 
"buyer_last_name" : "Surname", 
"description" : "", 
"paid_date" : "", 
"currency" : "GBP", 
"lang" : "en", 
"exchange_currency" : "", - converted currency (conversion of the sum and tax into another currency at current exchange rates)
"internal_note" : "", 
"invoice_template_id" : "1", 
"description_footer" : "", 
"description_long" : "", 
"from_invoice_id" : "" - invoice id, on which basis the invoice was generated (useful when generating a VAT invoice from Proforma invoice)
"positions":
   		"product_id" : "1", 
   		"name" : "InvoiceOcean Basic", 
   		"additional_info" : "", - additional information on invoice position 
   		"discount_percent" : "", - percentage discount (note: in order for the discount to be calculated, you need to set field 'show_discount' to 1 and before issuing check if in Account Settings, field: "How to calculate discount" is set to 'percentage from unit gross price')
   		"discount" : "", - amount discount (note: in order for the discount to be calculated, you need to set field 'show_discount' to 1 and before issuing check if in Account Settings, field: "How to calculate discount" is set to "amount")
   		"quantity" : "1", 
   		"quantity_unit" : "unit", 
   		"price_net" : "59,00", - if not entered it will be calculated
   		"tax" : "23", 
   		"price_gross" : "72,57", - if not entered it will be calculated
   		"total_price_net" : "59,00", - if not entered it will be calculated
   		"total_price_gross" : "72,57"
```

Field entries

Field: `kind`
```shell
	"vat" - VAT invoice
	"proforma" -  Proforma invoice
	"bill" - bill
	"receipt" - receipt
	"advance" - advance invoice
	"final" - final invoice
	"correction" - Credit Note
	"vat_mp" - MP invoice 
	"invoice_other" - other invoice 
	"vat_margin" - margin invoice
	"kp" - cash received
	"kw" - cash disbursed
	"estimate" - Estimate
```

Field: `lang`
```shell
	"pl" - Polish
	"en" - English
	"de" - German
	"fr" - French
	"cz" - Czech
	"ru" - Russian
	"es" - Spanish
	"it" - Italian
	"nl" - Dutch
	"hr" - Croatian
```


Field: `income`
```shell
	"1" - income invoice
	"0" - cost invoice
```

Field: `payment_type`
```shell
	"transfer" - transfer
	"card" - card
	"cash" -  cash
	"any_other_text_entry" 
```

Field: `status`
```shell
	"issued" - issued
	"sent" - sent
	"paid" - paid
	"partial" - partially paid
```

Field: `discount_kind` - discount kind
```shell
	"percent_unit" - calculated from the unit price
	"percent_total" - calculated from the total price
	"amount" - amount
```


<a name="clients"/>
##Clients

<a name="c1"/>
Clients list

```shell
curl "http://YOUR_DOMAIN.invoiceocean.com/clients.json?api_token=API_TOKEN&page=1"
```

<a name="c2"/>
Get selected client by ID

```shell
curl "http://YOUR_DOMAIN.invoiceocean.com/clients/100.json?api_token=API_TOKEN"
```

<a name="c3"/>
Adding clients

```shell
curl http://YOUR_DOMAIN.invoiceocean.com/clients.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "client": {
            "name": "Klient1",
            "tax_no": "5252445767",
            "bank" : "bank1",
            "bank_account" : "bank_account1",
            "city" : "city1",
            "country" : "",
            "email" : "bank1",
            "person" : "person1",
            "post_code" : "post-code1",
            "phone" : "phone1",
            "street" : "street1",
            "street_no" : "street-no1"
        }}'
```

<a name="c4"/>
Client update

```shell
curl http://YOUR_DOMAIN.invoiceocean.com/clients/111.json \
    -X PUT \
    -H 'Accept: application/json'  \
    -H 'Content-Type: application/json'  \
    -d '{"api_token": "API_TOKEN",
        "client": {
            "name": "Klient2",
            "tax_no": "52524457672",
            "bank" : "bank2",
            "bank_account" : "bank_account2",
            "city" : "city2",
            "country" : "PL",
            "email" : "bank2",
            "person" : "person2",
            "post_code" : "post-code2",
            "phone" : "phone2",
            "street" : "street2",
            "street_no" : "street-no2"
        }}'
```


<a name="products"/>
##Products

<a name="p1"/>
Products list

```shell
curl "http://YOUR_DOMAIN.invoiceocean.com/products.json?api_token=API_TOKEN&page=1"
```

<a name="p2"/>
Products list with storage quantities for a specific magazine

```shell
curl "http://YOUR_DOMAIN.invoiceocean.com/products.json?api_token=API_TOKEN&warehouse_id=WAREHOUSE_ID&page=1"
```

<a name="p3"/>
Get selected product by ID

```shell
curl "http://YOUR_DOMAIN.invoiceocean.com/products/100.json?api_token=API_TOKEN"
```

<a name="p4"/>
Get selected product by ID with storage quantity for a specific magazine

```shell
curl "http://YOUR_DOMAIN.invoiceocean.com/products/100.json?api_token=API_TOKEN&warehouse_id=WAREHOUSE_ID"
```

<a name="p5"/>
Adding products

```shell
curl http://YOUR_DOMAIN.invoiceocean.com/products.json \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json'  \
    -d '{"api_token": "API_TOKEN",
        "product": {
            "name": "PoroductAA",
            "code": "A001",
            "price_net": "100",
            "tax": "23"
        }}'
```

<a name="p6"/>
Product update

```shell
curl http://YOUR_DOMAIN.invoiceocean.com/products/333.json \
    -X PUT \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    -d '{"api_token": "API_TOKEN",
        "product": {
            "name": "PoroductAA2",
            "code": "A0012",
            "price_net": "102"
        }}'
```

<a name="payments"/>
##Payments

###Fields description
* **city** - City from the sender's address
* **client_id** - ID of the client who makes the payment
* **comment** - Comment for the client
* **country** - Country from the sender's address
* **currency** - Currency of the payment
* **department_id** - ID of the department that the client belongs to
* **description** - Payment description
* **email** - Email of the sender
* **first_name** - First name of the sender
* **generate_invoice** - If generate an invoice that would match the payment
* **invoice_city** - City of the generated invoice's address
* **invoice_comment** - Comment for the generated invoice
* **invoice_country** - Country of the generated invoice's address
* **invoice_id** - ID of the invoice being paid for
* **invoice_name** - Name of the client on the generated invoice
* **invoice_post_code** - Post code of the generated invoice's address
* **nvoice_street** - Street of the generated invoice's address
* **invoice_tax_no** - Tax no. on the generated invoice
* **last_name** - Last name of the sender
* **name** - Name of the sender
* **oid** - ID of the order that is paid for
* **paid** - If the payment is already paid <Boolean>
* **paid_date** - Date when the payment was made
* **phone** - Phone of the sender
* **post_code** - Post code from the sender's address
* **price** - Price of the product that was paid for
* **product_id** - ID of the product that was paid for
* **promocode** - Promocode that was used with the payment
* **provider** - Name of the payment provider (for online payments)
* **provider_response** - Response of the payment provider
* **provider_status** - Status of the payment according to the provider
* **provider_title** - Title of the payment provider
* **quantity** - Quantity of the item that was paid for
* **street** - Street from the sender's address
* **kind** - payment kind (where it comes from). In case of API it should be set to "api".

### Listing all payments

#### XML
    curl "http://YOUR_DOMAIN.invoiceocean.com/payments.xml?api_token=API_TOKEN"
    
#### JSON
    curl "http://YOUR_DOMAIN.invoiceocean.com/payments.json?api_token=API_TOKEN"

### Select payment using ID

#### XML
    curl "http://YOUR_DOMAIN.invoiceocean.com/payments/100.xml?api_token=API_TOKEN"
    
#### JSON
    curl "http://YOUR_DOMAIN.invoiceocean.com/payment/100.json?api_token=API_TOKEN"
    
### Adding new payment

#### Minimal JSON (recommended)
```shell
curl #{domain}/payments.json 
	-H 'Accept: application/json'  
	-H 'Content-Type: application/json'  
	-d '{
		"api_token": "#{api_token}",
		"payment": {	
			"name":"Payment 001",
			"price": 100.05,
			"invoice_id": null,
			"paid":true,
			"kind": "api"
	     	}
	     }'
```

#### Full JSON (recommended)
```shell
curl #{domain}/payments.json 
	-H 'Accept: application/json'  
	-H 'Content-Type: application/json'  
	-d '{
		"api_token": "#{api_token}",
		"payment": {	
			"city": null,
			"client_id":null,
			"comment":null,
			"country":null,
			"currency":"PLN",
			"department_id":null,
			"description":"abonament roczny",
			"email":"email@email.pl",
			"first_name":"Jan",
			"generate_invoice":true,			
			"invoice_city":"Warszawa",
			"invoice_comment":"",
			"invoice_country":null,
			"invoice_id":null,
			"invoice_name":"Company name",
			"invoice_post_code":"00-112",
			"invoice_street":"street 52",
			"invoice_tax_no":"5252445767",
			"last_name":"Kowalski",
			"name":"Plantnosc za produkt1",
			"oid":"",
			"paid":true,
			"paid_date":null,
			"phone":null,
			"post_code":null,
			"price":"100.00",
			"product_id":1,
			"promocode":"",
			"provider":"transfer",
			"provider_response":null,
			"provider_status":null,
			"provider_title":null,
			"quantity":1,
			"street":null,
			"kind": "api"
		}
	     }'
```

<a name="codes"/>
##Examples in PHP and Ruby

<https://github.com/radgost/fakturownia-api/blob/master/example1.php/>

<https://github.com/radgost/fakturownia-api/blob/master/example1.rb/>

Ruby Gem for InvoiceOcean.com integration: <https://github.com/kkempin/fakturownia/>
