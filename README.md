Ruby Lane Braintree Tcl SDK
===========================

This repository contains the tcl package, that enables integration to
the Braintree Gateway. All aspects have been based on the Ruby SDK,
version 2.38.0.

At the moment, this provides the gateway functionality for the Braintree
Marketplace solution, the full details of which are available at
https://developers.braintreepayments.com/javascript+ruby/guides/marketplace/overview.

To start using Braintree, you may sign up for a sandbox profile at
https://www.braintreepayments.com/get-started

Features Included
-----------------

This TCL package provides the following gateway methods:

* Client token generation
* Merchant account creation (for on-boarding sub merchants)
* Merchant account lookup
* Merchant account update
* Transaction 'sale'
* Transaction lookup


Features Not Yet Provided (planned)
--------------------------------------

* Transaction 'void'
* Transaction 'refund'
* Transaction 'submit for settlement' (post original sale)
* Transaction 'clone'


Dependencies
------------

This package has a dependency on the following packages:

* Tcl 8.6
* sha1
* tls
* [rl_http](https://github.com/RubyLane/rl_http)
* [rltest](https://github.com/RubyLane/rltest)
* [parse_args](https://github.com/RubyLane/parse_args)
* [tdom](https://github.com/RubyLane/tdom)

Examples
--------

To initialize the Braintree Gateway, create an object, passing through your
credentials:

~~~tcl
::rl_braintree create bt \
    -merchant_id    $your_merchant_id \
    -public_key     $your_public_key \
    -private_key    $your_private_key 
~~~

If you want to interact with the Braintree Sandbox environment, append
'-sandbox 1' to the above.

#Client Token Generation

~~~tcl
set token [bt generate_client_token]
~~~

#Merchant Accounts

###Main Parameters

* tos_accepted : An indication that the sub merchant has read, and
  agrees to the [Braintree Sub Merchant Terms Of
Service](https://developers.braintreepayments.com/javascript+ruby/guides/marketplace/overview#terms-of-service-accepted-parameter).
* master_merchant_account_id : Specifies the merchant account thet
  receives the service fees. This can be viewed in the Braintree Control
Panel.
* id : Specifies the id of the sub merchant account. If not provided,
  one will be automatically generated by Braintree.

###Individual Parameters

All sub merchant accounts must be associated to an individual. All
fields are required, except for:

* ssn : Braintree will attempt to retrieve the Social Security Number
  based on other information, if not provided. If given, this may be
either the full 9 digit number, or simply the last 4.
* phone

###Business Paramters

If the sub merchant is also registered as a Business, then these fields
should be provided. The following fields are mandatory:

* legal_name
* tax_id

All other fields, as shown in the example below, are optional for this
segment.

###Funding Paramters

* descriptor : optional. This sets the description that will appear on
  your bank statements. If not provided, Braintree will generate one
based on your name, business legal name or 'dba' name.
* funding_destination : required. Must be one of 'email' , 'mobile_phone' or 'bank'.
* email : required only if the funding destination is 'email'.
* mobile_phone : required only if the funding destination is
  'mobile_phone'.
* account_number : required only if the funding destination is 'bank'.
* routing_number : required only if the funding destination is 'bank'.

##Create (On-boarding)

A merchant account needs to be created for profiles associated with the
Marketplace.

~~~tcl
set result [bt merchant_account create {
    individual {
        first_name              "Jane"
        last_name               "Doe"
        email                   "jane@14ladders.com"
        phone                   "5553334444"
        date_of_birth           "1981-11-19"
        ssn                     "456-45-4567"
        address {
            street_address      "111 Main St"
            locality            "Chicago"
            region              "IL"
            postal_code         "60622"
        }
    }
    business {
        legal_name              "Jane's Ladders"
        dba_name                "Jane's Ladders"
        tax_id                  "98-7654321"
        address {
            street_address      "111 Main Str"
            locality            "Chicago"
            region              "IL"
            postal_code         "60622"
        }
    }
    funding {
        descriptor              "Blue Ladders"
        destination             "bank"
        email                   "funding@blueladders.com"
        mobile_phone            "5555555555"
        account_number          "1123581321"
        routing_number          "071101307"
    }
    tos_accepted                true
    master_merchant_account_id  "14ladders_marketplace"
    id                          "blue_ladders_store"
}]
~~~

##Update

To update an existing merchant account, simple change the applicable
field and pass the same data structure through. At this point, the 'id' of the sub
merchant account is required (unlike create where it is optional), to identify which profile is being updated.

~~~tcl
set result [bt merchant_account update {
    id "blue_ladders_store"
    ...
}]
~~~

##Lookup

The only mandatory field required is the sub merchant id:

~~~tcl
set result [bt merchant_account find { id "blue_ladders_store" }]
~~~

#Transaction

The absolute minimum attributes required, to create a transaction, are
an amount and a valid [payment method
nonce](https://developers.braintreepayments.com/javascript+ruby/sdk/overview/how-it-works#payment-method-nonce)

~~~tcl
set result [bt transaction sale {
    amount                  "100.00"
    payment_method_nonce    "nonce-from-the-client"
}]
~~~

To collect funds at the time of creating a transaction, you will need to
pass the option 'submit_for_settlement' as true.

**NOTE** at this point, this is required as the submit for settlement
method (after the initial creation) has not been finalised.

####Fuller example

~~~tcl
set result [bt transaction sale {
    amount                      "100.00"
    order_id                    "order_id"
    merchant_account_id         "a_merchant_account_id"
    payment_method_nonce        "nonce-from-the-client"
    customer {
        first_name              "Drew"
        last_name               "Smith"
        company                 "Braintree"
        phone                   "312-555-1234"
        fax                     "312-555-1235"
        website                 "http://www.example.com"
        email                   "drew@example.com"
    }
    billing {
        first_name              "Paul"
        last_name               "Smith"
        company                 "Braintree"
        street_address          "1 E Main St"
        extended_address        "Suite 403"
        locality                "Chicago"
        region                  "IL"
        postal_code             "60622"
        country_code_alpha2     "US"
    }
    shipping {
        first_name              "Jan"
        last_name               "Smith"
        company                 "Braintree"
        street_address          "1 E 1st St"
        extended_address        "Suite 403"
        locality                "Bartlett"
        region                  "IL"
        postal_code             "60103"
        country_code_alpha2     "US"
    }
    options {
        submit_for_settlement   true
    }
    channel                     "MyShoppingCartProvider"
}]
~~~

A full list of the transaction attributes can be found
[here](https://developers.braintreepayments.com/javascript+ruby/reference/response/transaction#transaction-attributes).

##Lookup

The only mandatory field required is the transaction id:

~~~tcl
set result [bt transaction find { id "braintree_generated_id" }]
~~~

License
-------

This package is licensed under the same terms as those defined by
Braintree.

See the file LICENSE for the information on the usage and redistribution of this file, and for a DISCLAIMER OF ALL WARRANTIES
