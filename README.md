# SSLCommerz

## install
```
composer require radon/sslcommerz
```

### publish
```
php artisan vendor:publish --provider=Radon\SslCommerz\SslCommerzServiceProvider
```
This command will create a `sslcommerz.php` file inside the `config` directory. Configure your parameters in your `.env` file
```
#sslcommerz
STORE_ID=your_store_id
STORE_PASSWORD=your_store_password
SUCCESS_URL=http://your-domain.at/success.php
FAIL_URL=http://your-domain.at/fail.php
CANCEL_URL=http://your-domain.at/cancel.php
IPN_URL=http://your-domain.at/ipn.php
SANDBOX_MODE=true
``` 

### Initiating a Payment session
This `initSession` will give you a gateway url. With this url you will be able to continue the payment through sslcommerz.
```php
$customer = new Customer('<<customer_name>>', '<<customer_email>>', '<<customer_phone>>');
$resp = Client::initSession($customer, '<<amount>>'); 
echo $resp->getGatewayUrl();
```
or with configuration
```php
$customer = new Customer('<<customer_name>>', '<<customer_email>>', '<<customer_phone>>');
$config[SessionRequest::EMI] = '0';
$resp = Client::initSession($customer, '<<amount>>', $config);
echo $resp->getGatewayUrl();
```

### Request for Validation
This `verifyOrder` method takes a `val_id` as parameter which you will get in the IPN request.

```php
$resp = Client::verifyOrder('<<order_number>>');
echo 'status: '.$resp->getStatus();
echo 'transaction: '.$resp->getTransactionId();
```

### IPN Listener (Step 4,5)
After filling the card information and submission in the sslcommerz window it will send a IPN notificaion to your
specified IPN url. To grab the notification use the following code. For more details [see here](https://developer.sslcommerz.com/docs.html)
 
```php
if(ipn_hash_varify(config('sslcommerz.store_password'))){
    $ipn = new IpnNotification($_POST);
    $val_id = $ipn->getValId();
    $transaction_id = $ipn->getTransactionId();
    $amount = $ipn->getAmount();
    $resp = Client::verifyOrder($val_id);
} 
```
