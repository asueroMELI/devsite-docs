# IPN Notifications

----[mla,mlb,mlc,mlm,mco,mlu]----
> WARNING
>
> Prerequisites
>
> * Have the [Checkout](https://www.mercadopago.com.ar/developers/es/guides/payments/web-payment-checkout/introduction) implemented.
------------
----[mpe]----
> WARNING
>
> Prerequisites
>
> * Have the [Checkout](https://www.mercadopago.com.ar/developers/en/guides/payments/web-checkout/introduction) implemented.
------------

**IPN** (Instant Payment Notification) is a notification sent from one server to another through an `HTTP POST` request informing your transactions.

In order to receive notifications about the events in your platform, you have to [previously configure an URL to which Mercado Pago has access.
](https://www.mercadopago.com.ar/herramientas/notificaciones)


## Events

We notify events related to your orders (`merchant_orders`), received chargebacks (`chargebacks`) or received payments (`payment`).

A `merchant_order` is an entity that groups payments as well as shipments. You will have to check the data of the orders notified to you.

Whenever an event related to any of the mentioned resources takes place, we will send you a notification using `HTTP POST` to the URL that you specified.

If your application is not available or takes too long to respond, Mercado Pago will retry sending the notification according to the following interval:

1. Retry after 5 minutes.
2. Retry after 45 minutes.
3. Retry after 6 hours.
4. Retry after 2 days.
5. Retry after 4 days.

Mercado Pago will notify to this URL whenever a resource is created or when orders or payment status are updated, with two parameters:

| Field 		| Description 				 |
| ---- 		| ---- 				 |
| `topic` | Identifies the type of resource. It may be `payment`, `chargebacks` or `merchant_order` |
| `id` | A unique identification of the notified resource. |


Example: If you have configured the URL:  `https://www.yoursite.com/notifications`, you will receive payment notifications as follows:  `https://www.yoursite.com/notifications?topic=payment&id=123456789`

## What should I do after receiving a notification?

When you receive a notification in your platform, Mercado Pago awaits a response to validate that you received it correctly. To do this, you have to send a response with a `HTTP STATUS 200 (OK)` or `201 (CREATED)`.

Note that this communication is made exclusively between MercadoPago’s servers and your server, so there will be no physical user viewing any kind of result.

After that, you will be able to get full information about the notified resource by accessing the corresponding API at https://api.mercadopago.com/:

Type               | URL                                                         | Documentation
------------------ | ----------------------------------------------------------- | --------------------
payment            | /v1/payments/[ID]?access\_token=[ACCESS\_TOKEN] | [see documentation](https://www.mercadopago.com.ar/developers/en/reference/payments/_payments_id/get/)
chargebacks    	   | /v1/chargebacks/[ID]?access\_token=[ACCESS\_TOKEN]| -
merchant_orders    | /merchant\_orders/[ID]?access\_token=[ACCESS\_TOKEN]           | [see documentation](https://www.mercadopago.com.ar/developers/en/reference/merchant_orders/_merchant_orders_id/get/)


### Implement the notification receiver using the following code as example:

```php
<?php
	MercadoPago\SDK::setAccessToken("ENV_ACCESS_TOKEN");

	$merchant_order = null;

	switch($_GET["topic"]) {
		case "payment":
			$payment = MercadoPago\Payment::find_by_id($_GET["id"]);
			// Get the payment and the corresponding merchant_order reported by the IPN.
			$merchant_order = MercadoPago\MerchantOrder::find_by_id($payment->order_id;
		case "merchant_order":
			$merchant_order = MercadoPago\MerchantOrder::find_by_id($_GET["id"]);
	}

	$paid_amount = 0;
	foreach ($merchant_order->payments as $payment) {	
		if ($payment['status'] == 'approved'){
			$paid_amount += $payment['transaction_amount'];
		}
	}
	
	// If the payment's transaction amount is equal (or bigger) than the merchant_order's amount you can release your items
	if($paid_amount >= $merchant_order->total_amount){
		if (count($merchant_order->shipments)>0) { // The merchant_order has shipments
			if($merchant_order->shipments[0]->status == "ready_to_ship") {
				print_r("Totally paid. Print the label and release your item.");
			}
		} else { // The merchant_order don't has any shipments
			print_r("Totally paid. Release your item.");
		}
	} else {
		print_r("Not paid yet. Do not release your item.");
	}
	
?>
```

> To get your `ACCESS_TOKEN`, check the [Credentials](https://www.mercadopago.com.ar/account/credentials?type=basic) section.
