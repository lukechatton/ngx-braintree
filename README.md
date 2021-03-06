<h1>Integrating Braintree in Angular applications</h1>

This module integrates the Braintree Dropin UI integration (v3) with your Angular 4.x and 5.x applications. The integration aims at componentizing the Braintree-Angular integration so that you can just use the component `<ngx-braintree></ngx-braintree>` anywhere in your application and you are good to go. 

![Demo](https://srikanth.onl/wp-content/uploads/2017/12/demo.gif)

## Usage

> Note: This is not an official Braintree Angular component.

First, if your application is in Angular 5.x, install `ngx-braintree` by issuing the following command:

> npm install ngx-braintree --save

If your application is an Angular 4.x application, install `ngx-braintree` by issuing the following command:

> npm install ngx-braintree@a4

After the above step is done, import it into your module:

> import { NgxBraintreeModule } from 'ngx-braintree';

`ngx-braintree` uses `HttpClientModule`, so import that as well:

> import { HttpClientModule } from '@angular/common/http';

Now in the imports section of @NgModule add these two lines `NgxBraintreeModule` and `HttpClientModule` as shown below:

>  imports: [ NgxBraintreeModule, HttpClientModule ]

Now that you have finished all the above steps, you are now ready to use the ngx-braintree component in your application. Where ever you want the Braintree Dropin UI in your application, you can use the `<ngx-braintree></ngx-braintree>`component as shown below:

	<ngx-braintree 
		[clientTokenURL]="'api/braintree/getclienttoken'" 
		[createPurchaseURL]="'api/braintree/createpurchase'" 
		(paymentStatus)="onPaymentStatus($event)">
	</ngx-braintree>
	
**clientTokenURL** – is **YOUR** server-side API GET URL. 
This is YOUR server-side API GET method which calls Braintree and gets the clientToken for the Drop-in UI. A sample server API method that gives the clientToken is as shown below (.NET Code). `ngx-braintree` starts displaying the UI as soon as it receives the clientToken that your server provides. For more information read the Braintree Server API section below.

		[Route("api/braintree/getclienttoken")]
        public HttpResponseMessage GetClientToken()
        {
            var gateway = new BraintreeGateway
            {
                Environment = Braintree.Environment.SANDBOX,
                MerchantId = "your_braintree_merchant_id",
                PublicKey = "your_braintree_public_key",
                PrivateKey = "your_braintree_private_key"
            };

            var clientToken = gateway.ClientToken.Generate();
            HttpResponseMessage response = Request.CreateResponse(clientToken);
            return response;
        }

**createPurchaseURL** – is **YOUR** server-side API POST URL.
This is YOUR server-side API POST method which is called when the user clicks Pay. `ngx-braintree` will post the payment method nonce to the URL you provide through which you process the payment from your server and return the response. A sample server API POST method is as shown below (.NET Code). 

**Note: It is important to set your POST method's parameter as `Nonce nonce` as shown below.**

        public class Nonce
        {
            public string nonce;

            public Nonce(string nonce)
            {
                this.nonce = nonce;
            }
        }

		[Route("api/braintree/createpurchase")]
        public HttpResponseMessage Post([FromBody]Nonce nonce)
        {
            var gateway = new BraintreeGateway
            {
                Environment = Braintree.Environment.SANDBOX,
                MerchantId = "your_braintree_merchant_id",
                PublicKey = "your_braintree_public_key",
                PrivateKey = "your_braintree_private_key"
            };

            var request = new TransactionRequest
            {
                Amount = 1000.00M,
                PaymentMethodNonce = nonce.nonce,
                Options = new TransactionOptionsRequest
                {
                    SubmitForSettlement = true
                }
            };

            Result<Transaction> result = gateway.Transaction.Sale(request);
            HttpResponseMessage response = Request.CreateResponse(result);
            return response;
        }

**paymentStatus** - is the event that you should listen to. The `paymentStatus` event is emitted when a payment process finishes. The event emits the response that your purchase URL API method (createPurchaseURL) returns. Returning the same response, helps you in accessing the response object on the client side and also helps you make decisions whether to redirect user to the payment confirmation page (if the payment succeeded) or to do something else if anything went wrong.

> Make sure the values of **clientTokenURL** and **createPurchaseURL** are enclosed in single quotes

<h1>Optional configurations for `ngx-braintree` component</h1>

The `ngx-braintree` component can be optionally configured by providing the following inputs to the component.

1. **[buttonText]**: This allows you to configure the text of the pay button. The default text on the button is Buy. If you want to have a custom text such as 'Pay' or 'Pay Now', you can configure it. Pass the text you desire as the value of the input property as shown below:

		<ngx-braintree 
			[clientTokenURL]="'api/braintree/getclienttoken'" 
			[createPurchaseURL]="'api/braintree/createpurchase'" 
			(paymentStatus)="onPaymentStatus($event)"
			[buttonText]="'Pay'">
		</ngx-braintree>

	> Make sure the value of **buttonText** is enclosed in single quotes.
2. **[allowChoose]**: This provides you the ability to configure whether to let the user choose another way to pay after he has entered the payment details. Pass true or false as the value of the input property as shown below. The default will be false, if you don't specify this configuration.

		<ngx-braintree 
		    [clientTokenURL]="'api/braintree/getclienttoken'" 
		    [createPurchaseURL]="'api/braintree/createpurchase'"
		    (paymentStatus)="onPaymentStatus($event)"
		    [buttonText]="'Pay'"
		    [allowChoose]="true">
		</ngx-braintree>
		
	This is a two step process that Braintree supports. You can configure ngx-braintree to make it work in the following way:

	1. If **[allowChoose]** is set to true, as soon as the user enters payment details and clicks Pay, user will be shown another UI where he can opt to change his payment details by choosing another payment method or just click Pay again as shown below: <br />![Two step process](https://srikanth.onl/wp-content/uploads/2017/12/twostep.gif)	
	2. If **[allowChoose]** is set to false, it will only be a one step process and the user is not given any option to change his payment details and the payment process will continue as soon as he clicks Pay as shown below. This is the default setting of `ngx-braintree` component. <br />![One step process](https://srikanth.onl/wp-content/uploads/2017/12/onestep.gif)
		
<h1>Braintree Server API</h1>

As mentioned above, along with the client side work (which `ngx-braintree` component fully takes care of), Braintree also requires us to write two server side API methods. To successfully use the `ngx-braintree` component, a simple API with two methods is required (.NET code for those two methods is shown above). One method's URL is the value for the **clientTokenURL** and other method's URL is the value for the **createPurchaseURL** properties of the `ngx-braintree` component. These API methods can be developed very easily on any server platform by visiting the following link https://developers.braintreepayments.com/start/hello-server/dotnet

<h1>Issues</h1>

Please report any issues/feature requests here: https://github.com/srikanthonl/ngx-braintree/issues

<h1>More Information</h1>

For more information please visit

https://srikanth.onl/integrating-braintree-with-angular-applications/
