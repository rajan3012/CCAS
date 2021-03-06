# Rest API development < Nodejs + Express + Axios + Mocha > 

# Central Cascade Automative Sales 

Our project manager has come to us with a brand new product-- an exciting online ordering
system for custom vehicle orders. Since we have a lot of customers who like to choose options
we don’t necessarily have in stock, we have decided to build an interface through which the
customers can place their orders. This user client will require a backend service to
communicate with. The purpose of this service will be to take an order and send it off to the
appropriate supplier for delivery.

## Project Workflow
	1.	Customer provides {customer_id, make, model, package} as request to Central Cascade
		Automotive Sales(CCAS). 
		CCAS maintains Customer Schema as described below.

	2.	CCAS validates customer_id against stored customer information.
		If the customer_id is invalid -> stop request.
		If the customer's mentioned Shipping address in not in USA, decline service.

	3.	If the customer_id and make are valid, do the following depending on the “make”

			1.	If make is “Acme Autos” or "ACME"
				-	Send request to the Acme API with {api_key, model, package}
				-	Only a valid api_key guarantees response from the Acme API
				-	Acme sends back {order_id} as response

			2.	If make is “Rainier” or "Rainier Transportation Solution" or "RTS"
				-	Send token request to the RTS API with {storefront}
				-	RTS responds with {nonce_token}
				-	CCAS posts request to RTS with {nonce_token, model, package}
				-	RTS replies back with {order_id} as response if nonce_token is valid

	4.	Store order details for internal use:
		{order_id, customer_id, supplier_id, make, model, package}
			-	Supplier_id helps identify supplier. order_id and customer_id cannot uniquely 
				identify the supplier. Eg: Both Acme and Rainier could have the same order_id
			-	Make is unique in our case, but by convention it is better to use IDs than strings.

	5.	Inform customer of successful request and provide json object link for that order_id

## Database Model


**1.	Customer**

{ customerId, name.firstName, name.LastName, address.city, address.state, address.country }

**2.	Orders**

{ orderId, customerIid, supplierIid, make, model, package}

NOTE : here orderId represents the orderId returned by the Supplier API on successful order placement

**3.	Supplier**

{ supplierId, make, api_key, storefront, token }


*All schemas support GET, POST, PUT and DELETE operations*

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

* Node.js - v6.9.1
* Express.js
* MongoDB

**Packages used:**

* Nodemon - For seamless building of the project
* Mongoose - Facilitate MongoDB communication
* Lodash - JavaScript utility functions
* Axios - Handling HTTP requests
* Mocha - Testing environment
* Chai - Assertion library
* Chai HTTP - HTTP integration testing with Chai assertions

### Installing

Clone the project using :
```
git clone https://github.com/rajan3012/CCAS.git
```

Do a ```npm install``` to get all the required packages mentioned in package.json

### Run the servers

First step would be to start a MongoDB instance. Run ```mongod``` in a terminal

Open the servers in different terminals. Each server runs on an individual terminal

Start the servers
```
npm start ccas
npm start acme
npm start rts
```

### Running the project:

Once you have all the servers up and running, let's begin to use the CCAS API!

**Customer creation:**

-- Open Postman 

-- Do a POST request on the **/customer** endpoint:
```
http://localhost:3000/customer
```
--Go to Body in postman, select x-www-form-urlencoded and enter:
			
		name.firstName = Rajan
		name.lastName = Saw
		address.city = Eugene
		address.state = Oregon
		address.country = USA

-- Hit send. You should get back a json object. Something like this:
```
{
	name: {
		firstName: 'Rajan', lastName: 'Saw' },
		address: { 
			city: 'Eugene', 
			state: 'Oregon', 
			country: 'USA' 
		},
	_id: 5a602b8295701c26d5da35ce,
	__v: 0
}

```

We have our first customer!

You can do go a GET at the customer endpoint to see all customers in the database

Let's add two suppliers : ACME Autos and Rainier Transportation Solutions


**Supplier creation:**

-- In postman open 
```
http://localhost:3000/supplier
```
-- POST request at the **/supplier** endpoint and enter:
	
	supplierId: 111
	make: ACME Autos
	api_key: cascade.53bce4f1dfa0fe8e7ca126f91b35d3a6


-- Hit send. ACME has been added! 

![alt text](https://media.giphy.com/media/KqItY5h1jmP0k/200.gif)


-- Now let's add RTS information:


	supplierId: 222
	make: RTS 
	api_key: ccas-bb9630c04f 
	
-- Hit send again, and we should have something like this:
```
[
 {
	"_id": "5a60131fe720ba1cb706c26c",
	"supplierId": 111,
	"make": "ACME",
	"api_key": "cascade.53bce4f1dfa0fe8e7ca126f91b35d3a6",
	"__v": 0
 },
 {
	"_id": "5a60144ce720ba1cb706c26d",
	"supplierId": 222,
	"make": "RTS",
	"storefront": "ccas-bb9630c04f",
	"__v": 0
 } 
]
```

We have customers and suppliers! Let's place some orders!

**Order creation:**
```
http://localhost:3000/order
```
-- Select POST, select x-www-form-urlencoded:

-- Enter the following details in the body:

	customerId: // this is _id of the customer in our database
	make: //ACME or RTS
	model: //some model eg:[olympic, roadrunner]
	carPackage: //some package eg:[lite, extreme, mtn]


-- Hit send and if you see something like this...:
```
{
	"message": "Order placed successfully",
	"order_details": {
			"_id": "5a602baa95701c26d5da35cf",
			"customerId": "5a602b8295701c26d5da35ce",
			"make": "RTS",
			"model": "olympic",
			"carPackage": "mtn",
			"supplierId": 222,
			"orderId": 4568140,
			"__v": 0
	},
	"link": "http://localhost:3000/order/order-4568140"
}
```

...you have successfully placed an order with CCAS!

Once you place the order, you can view it's details using the link mentioned as part of the JSON object.



**Making the CCAS API secure:**


For security purposes, only CCAS customer service representatives can view all the orders.

To view all orders, use the **/orders** endpoint :
```
http://localhost:3000/orders
```

**ACCESS DENIED!**


![alt text](https://media.giphy.com/media/11QzYvIrd3cZpe/giphy.gif)



Provide the PIN as parameters. Add pin=1123 to get access.
Like this:
```
http://localhost:3000/orders?pin=1123
```

You should get back an Orders Report

## Other REST functionalities ##

You can update and delete objects in the Customer, Supplier and Order collection 
using the \_id generated for these objects:

For example:

-- To update an object, do a PUT request against the endpoint and pass the \_id along. This way: 
```
http://localhost:3000/customer/5a601be5e720ba1cb706c26e
```

And enter the key and their values in the body with x-www-form-urlencoded.

-- Similary to delete an object, do a DELETE request.



## Running the tests

To run tests, execute:

```
npm run test
```

### Test endpoints:

* Customer
	
	- GET a customer
	- POST a customer
	- PUT a customer
	- DELETE a customer

Mocha + Chai used

## Authors

* **Rajan Sawhney** [rajansawhney](https://github.com/rajansawhney)


## License

This readme is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

