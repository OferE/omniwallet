## What is Mastercoin?

[from wiki.mastercoin.org]
Mastercoin is both a new type of currency (MSC) and a platform. It is a new protocol layer 
running on top of bitcoin like HTTP runs on top of TCP/IP. Its purpose is to build upon the 
core Bitcoin protocol and add new advanced features, with a focus on a straight-forward and 
easy to understand implementation which allows for analysis and its rapid development. 

## What is Omniwallet?

Omniwallet is a new type of web wallet, that combines security, ease of use, multi-currency support, and is completely open source from the ground up (up to the deplomeny scripts!)

It currently supports Bitcoin and Mastercoin, and will support Mastercoin-derived currency in the future. In addition, support for other blockchains is a high priority for us - you will be able to store Litecoins, Peercoins, and other alts on the same highly secure web wallet (See [running bounty for Peercoin integration](http://peer4commit.com/projects/17)).

You can checkout the test builds at [test.omniwallet.org](https://test.omniwallet.org/) (please note that this is not production ready, only use small amounts of BTC and MSC!)

### Best in class security

1. Private keys are never sent to the server except in an encrypted form.
2. Private keys are always backed up (encrypted), so even if the server goes offline or is compromised, you can always migrate to another wallet
3. Everything is open source from the ground up!
4. You can deploy the Omniwallet on your own server and host your own instance, or use one of the service providers that will host it for you – your money, your choice.

### Baked in usability

1. No software to download or install, no blockchain to synchronize and verify – it just works, lightning fast.
2. Carefully planned layout, with common operations emphasized
3. Beautiful and intuitive User Interface
4. Special care is taken to smooth out operational edge cases to prevent mistakes and ensure a painless experience for the user

### Multi-currency support

1. Omniwallet comes with a pre-built support for Bitcoin, Mastercoin and Test Mastercoins
2. Native support for Smart Property and User-Generated Currencies
3. Alt-coin support – the Omniwallet will support coins that go outside of the usual Mastercoin ecosystem, such as Litecoin, Peercoin and more.

## Setup

Install sx
```
sudo apt-get install git build-essential autoconf libtool libboost-all-dev pkg-config libcurl4-openssl-dev libleveldb-dev libzmq-dev libconfig++-dev libncurses5-dev
wget http://sx.dyne.org/install-sx.sh
sudo bash ./install-sx.sh
```
update ~/.sx.cfg with an obelisk server details.  Don't have one already set up?  Here's how to build one on Rackspace: https://gist.github.com/curtislacy/8424181
```
# ~/.sx.cfg Sample file.
service = "tcp://162.243.29.201:9091"
```
Make sure you have python libraries installed - note that we use ``apt-get`` to install python-git.  Pip installs an older, stable version, and we need things that start in beta version 0.3.2.
```
sudo apt-get install git python-simplejson python-git python-pip libffi-dev
sudo pip install -r requirements.txt
```
Install nginx, and drop in the config included with this codebase.
```
sudo apt-get install uwsgi uwsgi-plugin-python
sudo -s
nginx=stable # use nginx=development for latest development version
add-apt-repository ppa:nginx/$nginx
apt-get update 
apt-get install nginx
exit
sudo cp etc/nginx/sites-available/default /etc/nginx/sites-available
```
Find this section near the beginning of /etc/nginx/sites-available/default:
```
        ## Set this to reflect the location of the www directory within the omniwallet repo.
        root /home/cmlacy/omniwallet/www/;
        index index.html index.htm;
```
Change the ``root`` directive to reflect the location of your omniwallet codebase (actually the www directory within that codebase).

Make sure you have uglifyjs (Note that there are a couple flavors of this available - you need the ``uglifyjs`` executable, which is included in the ``uglify-js`` NPM module - NOT the ``uglifyjs`` module!
```
sudo npm install -g uglify-js
```
Run npm install
```
npm install
```
Create the parsed blockchain data directory
```
sudo mkdir /var/lib/omniwallet
sudo chown {user who will run omniwallet} /var/lib/omniwallet
```

## Running

Start nginx by running:
```
sudo service nginx start
```
Using the config included, nginx will launch an HTTP server on port 80.

Set an environment variable containing a secret passphrase - this is used to generate salts for indivdual user IDs, and it needs to be both secret AND not change.
```
export OMNIWALLET_SECRET="DontTellAnyoneThis"
```
Start the blockchain parser and python services by running:

```
app.sh
```

This will create a parsing & validation work area in /tmp/omniwallet, and begin parsing the blockchain using the server listed in your .sx.cfg file (see above).

## Development

Most of the HTML in /www is checked in, however three files are generated at install:
```
www/Address.html
www/index.html
www/simplesend.html
```
These are generated by the gen_www.sh script as a part of "npm install".  If you need to regenerate them, and do not wish to run a full "npm install" (though that's pretty fast), you can also run:
```
grunt build
```
If you install the development dependencies (``npm install --development``), you'll also be able to use the ``serve-static.js`` script, which can save you the effort of running nginx if you're just doing development on the static HTML pages.

## Signing with your PGP key

Signing your commits with a PGP key is always appreciated.
1. Generate a key: http://stackoverflow.com/a/16725717/364485
2. Sign your commit: ``git commit -S`` (Works for merges too, don't need to sign every commit, just the last one before you push something up.
3. Check the signature on your commit: https://github.com/mastercoin-MSC/omniwallet/commit/d05dd949acb7234843d0e32b50c12a3556b8444b

## READ API (HTTP GET)

### Get currencies and page counts
HTTP GET ``/v1/transaction/values.json``
Returns:
```
[
	{
		"accept_pages": 0, 
		"currency": "MSC", 
		"name": "Mastercoin", 
		"name2": "", 
		"pages": 355, 
		"sell_pages": 0, 
		"trend": "down", 
		"trend2": "rgb(13,157,51)"
	}, 
	{
		"accept_pages": 7, 
		"currency": "TMSC", 
		"name": "Test MSC", 
		"name2": "", 
		"pages": 126, 
		"sell_pages": 4, 
		"trend": "up", 
		"trend2": "rgb(212,48,48)"
	}
]
```

### Get coin values
HTTP GET ``/v1/values/<coin symbol>.json``
Returns:
```
[
  {
    "price": 0.06865699418321376, 
    "symbol": "MSC"
  }
]
```

### Get coin historic values
HTTP GET ``/v1/values/history/<coin symbol>.json``
Returns:
```
[
  { 
    "timestamp": 1396571412.478132, 
    "value": {
      "price": 0.06865699418321376, 
      "symbol": "MSC"
    }
  }, 
  {
    "timestamp": 1396571448.99139, 
    "value": {
      "price": 0.06865699418321678, 
      "symbol": "MSC"
    }
  }
]

```

### Get Smart Property details
HTTP GET ``/v1/property/<property ID>.json``
Returns:
```
[ { baseCoin: '00',
    block: '293180',
    color: 'bgc-new',
    currencyId: 0,
    currency_str: 'Smart Property',
    dataSequenceNum: '01',
    details: 'unknown',
    ecosystem: '02',
    formatted_amount: 0,
    formatted_ecosystem: 2,
    formatted_previous_property_id: 0,
    formatted_property_type: 1,
    formatted_transactionType: 50,
    formatted_transactionVersion: 0,
    from_address: '1471EHpnJ62MDxLw96dKcNT8sWPEbHrAUe',
    icon: 'unknown',
    index: '486',
    invalid: false,
    method: 'multisig',
    numberOfProperties: '500',
    previous_property_id: '00000000',
    propertyCategory: 'Coupons, Gifts',
    propertyData: 'Each promo coupon allows one free redemption on the Mastercoin faucet.',
    propertyName: 'Mastercoin Faucet Promo Coupon',
    propertySubcategory: 'Web',
    propertyUrl: 'mastercoin-faucet.com/redeem-coupon',
    property_type: '0001',
    to_address: 'unknown',
    transactionType: '0032',
    transactionVersion: '0000',
    tx_hash: '331e4e204f9200a37fb5ea3364bfe52c33bfda3a1f0064d66e1b55e5faa03ba6',
    tx_method_str: 'multisig',
    tx_time: '1396151474000',
    tx_type_str: 'Fixed property creation',
    update_fs: false } ]
```

### Get system version information
HTTP GET ``/v1/system/revision.json``
Returns:
```
{
	"commit_hexsha": "c030d61e6e7d91c2149a7af8af6bca70e7c0638f", 
	"commit_time": "20140131", 
	"last_block": 284634, 
	"last_parsed": "07 Feb 2014 15:48:39 GMT", 
	"url": "https://github.com/grazcoin/mastercoin-tools/commit/c030d61e6e7d91c2149a7af8af6bca70e7c0638f"
}
```

### Get hosted wallet statistics
HTTP GET ``/v1/system/stats.json``
Returns:
```
{
	"amount_of_addresses_managed": 22,
	"amount_of_transactions": 131, 
	"amount_of_wallets": 29
}
```

### Get a page of transaction & offer information
HTTP GET ``/v1/transaction/general/{Currency}_{4 digit page}.json``
Returns:
```
[
	{
		"amount": "00000000004c4b40", 
		"baseCoin": "00", 
		"block": "284632", 
		"color": "bgc-done", 
		"currencyId": "00000002", 
		"currency_str": "Test Mastercoin", 
		"dataSequenceNum": "01", 
		"details": "1AxWcEqfV7FQ6qkZRL1PeHZ1EDGtpuZHCH", 
		"formatted_amount": "0.05", 
		"from_address": "1BxtgEa8UcrMzVZaW32zVyJh4Sg4KGFzxA", 
		"icon": "simplesend", 
		"icon_text": "Simple send", 
		"index": "266", 
		"invalid": false, 
		"method": "multisig", 
		"to_address": "1AxWcEqfV7FQ6qkZRL1PeHZ1EDGtpuZHCH", 
		"transactionType": "00000000", 
		"tx_hash": "cf919a86223e27ea49af5f5ec62cd5a8971f4ceaf0bab5a97c0e9849abf21774",
		"tx_method_str": "multisig", 
		"tx_time": "1391784042000", 
		"tx_type_str": "Simple send"
	}, 
	...
]
```



### Get detailed information about a particular offer
HTTP Get ``/v1/exchange/offers/offers-{offer ID}.json``
Returns:
```
[
	{
		"amount": "0000000077359400", 
		"baseCoin": "00", 
		"bitcoin_required": "2e-05", 
		"block": "267568", 
		"btc_offer_txid": "unknown", 
		"color": "bgc-expired", 
		"currencyId": "00000002", 
		"currency_str": "Test Mastercoin", 
		"dataSequenceNum": "01", 
		"details": "unknown_price", 
		"formatted_amount": "20.0", 
		"formatted_amount_accepted": 20.0, 
		"formatted_amount_bought": "0.0", 
		"formatted_amount_requested": "20.0", 
		"formatted_price_per_coin": "0.000001", 
		"from_address": "1EdAjiApS5cCpHdH4RKPMab1xmMVRWjLvk", 
		"icon": "sellaccept", 
		"icon_text": "Payment expired", 
		"index": "274", 
		"invalid": false, 
		"method": "multisig", 
		"payment_done": false, 
		"payment_expired": true, 
		"payment_txid": "Not available", 
		"sell_offer_txid": "02c300afb5c776d6013ba8833f4986f093e516c7511808146287b59689346596", 
		"status": "Expired", 
		"to_address": "13NRX88EZbS5q81x6XFrTECzrciPREo821", 
		"transactionType": "00000016", 
		"tx_hash": "33644e6f24b29e1ef170d78ff04eab6f7e19368908edc6d477f9902697a71d67", 
		"tx_method_str": "multisig", 
		"tx_time": "1383423463000", 
		"tx_type_str": "Sell accept", 
		"update_fs": true
	}
]
```

### Get detailed information about a particular transaction
HTTP GET ``/v1/transaction/tx/{transaction ID}.json``
Returns:
```
[
	{
		"amount": "00000000004c4b40", 
		"baseCoin": "00", 
		"block": "283922", 
		"color": "bgc-new", 
		"currencyId": "00000002", 
		"currency_str": "Test Mastercoin", 
		"dataSequenceNum": "01", 
		"details": "19TRR5mBqiV1ZtmbhYmTcCfxvayk8esrfF", 
		"formatted_amount": "0.05", 
		"from_address": "1HG3s4Ext3sTqBTHrgftyUzG3cvx5ZbPCj", 
		"icon": "simplesend", 
		"icon_text": "Simple send (1 confirms)", 
		"index": "68", 
		"invalid": false, 
		"method": "multisig", 
		"to_address": "19TRR5mBqiV1ZtmbhYmTcCfxvayk8esrfF", 
		"transactionType": "00000000", 
		"tx_hash": "71b7f453d43ef2d56c004b21ce5bedf1f3f2e05c6ce7464ebbc1c10df421eeeb", 
		"tx_method_str": "multisig", 
		"tx_time": "1391417992000", 
		"tx_type_str": "Simple send"
	}
]
```

## EDIT API (HTTP POST)

### Find out the balance information for a given address.
```
var dataToSend = { addr: address };
$.post('/v1/address/addr/', dataToSend, function ( result ) { 
	console.log( result.data )
}).fail( function() {} );
```
Yields:
```
{
	"0": {
		"accept_transactions": [], 
		"balance": "2.21965319", 
		"bought_transactions": [], 
		"exodus_transactions": [], 
		"offer_transactions": [], 
		"received_transactions": [
			{
				"amount": "000000000d3aec07", 
				"baseCoin": "00", 
				"bitcoin_amount_desired": "000000", 
				"block": "284544", 
				"block_time_limit": "", 
				"color": "bgc-new", 
				"currencyId": "00000001", 
				"currency_str": "Mastercoin", 
				"dataSequenceNum": "6c", 
				"details": "1AxWcEqfV7FQ6qkZRL1PeHZ1EDGtpuZHCH",
				"formatted_amount": "2.21965319", 
				"from_address": "1MaStErt4XsYHPwfrN9TpgdURLhHTdMenH", 
				"icon": "simplesend", 
				"icon_text": "Simple send (1 confirms)",
				"index": "449", 
				"invalid": false, 
				"method": "basic", 
				"to_address": "1AxWcEqfV7FQ6qkZRL1PeHZ1EDGtpuZHCH",
				"transactionType": "00000000", 
				"tx_hash": "e13e3018461a297e2ce3d2681d60e3f120efababc117fa9a40785029fb2d2282", 
				"tx_method_str": "basic", 
				"tx_time": "1391730636000", 
				"tx_type_str": "Simple send"
			}
		], 
		"sent_transactions": [], 
		"sold_transactions": [], 
		"total_bought": "0.0", 
		"total_exodus": "0.0", 
		"total_received": "2.21965319", 
		"total_sell_accept": "0.0", 
		"total_sell_offer": "0.0", 
		"total_sent": "0.0", 
		"total_sold": "0.0"
	}, 
	"1": {
		"accept_transactions": [], 
		"balance": "0.0", 
		"bought_transactions": [], 
		"exodus_transactions": [], 
		"offer_transactions": [], 
		"received_transactions": [], 
		"sent_transactions": [], 
		"sold_transactions": [], 
		"total_bought": "0.0", 
		"total_exodus": "0.0", 
		"total_received": "0.0", 
		"total_sell_accept": "0.0", 
		"total_sell_offer": "0.0", 
		"total_sent": "0.0", 
		"total_sold": "0.0"
	}, 
	"address": "1AxWcEqfV7FQ6qkZRL1PeHZ1EDGtpuZHCH", 
	"balance": [
		{
			"symbol": "MSC", 
			"value": "2.21965319"
		}, 
		{
			"symbol": "TMSC", 
			"value": "0.0"
		}, 
		{
			"symbol": "BTC", 
			"value": "0.57421089"
		}
	]
}
```

### Sending coins to an address:

#### Validate the source address:
```
var dataToSend = { addr: from_addr };
$.post('/v1/transaction/validateaddr/', dataToSend, function (data) {}).fail( function() {} );
```
``data`` will contain one of:

| Value                      | Meaning                  |
| -------------------------- | ------------------------ |
| ``{ "status": "OK" }``     | Address is a valid source |
| ``{ "status": "invalid pubkey" }``     | Public key on the blockchain for that address is invalid. |
| ``{ "status": "missing pubkey" }``     | No public key exists on the blockchain for this address.  Usually this means that the address has not yet sent any coins anywhere else. |
| ``{ "status": "invalid address" }``     | This address just isn't valid. |

#### Encode and sign the transaction:
```
var dataToSend = { 
	from_address: from_address, 
	pubKey: pubKey,
	to_address: to_address, 
	amount: amount, 
	currency: currency, 
	fee: fee, 
	marker: marker
};
$.post('/v1/transaction/send/', dataToSend, function (data) {

	//data should have fields sourceScript and transaction
	$('#sourceScript').val(data.sourceScript);
	$('#transactionBBE').val(data.transaction);

}).fail(function () {} );
```

#### Actually push up the transaction:
```
var rawTx = Crypto.util.bytesToHex(sendTx.serialize());
var dataToSend = { signedTransaction: rawTx };
$.post('/wallet/pushtx/', dataToSend, function (data) {}).fail( function() {} );
```
No return value in ``data``.

### Make a trade offer:

#### Validate the source address:
```
var dataToSend = { addr: from_addr };
$.post('/wallet/validateaddr/', dataToSend, function (data) {}).fail( function() {} );
```
``data`` will contain one of:

| Value                      | Meaning                  |
| -------------------------- | ------------------------ |
| ``{ "status": "OK" }``     | Address is a valid source |
| ``{ "status": "invalid pubkey" }``     | Public key on the blockchain for that address is invalid. |
| ``{ "status": "missing pubkey" }``     | No public key exists on the blockchain for this address.  Usually this means that the address has not yet sent any coins anywhere else. |
| ``{ "status": "invalid address" }``     | This address just isn't valid. |

#### Encode the trade offer:
```
var dataToSend = { seller: from_address, pubKey: pubKey, amount: amount, price: price, min_buyer_fee: min_buyer_fee, fee: fee, blocks: blocks, currency: currency };
$.post('/v1/transaction/sell/', dataToSend, function (data) {

	//data should have fields sourceScript and transaction\
	$('#sourceScript').val(data.sourceScript);
	$('#transactionBBE').val(data.transaction);

}).fail(function () {} );
```

#### Actually push up the transaction:
```
var rawTx = Crypto.util.bytesToHex(sendTx.serialize());
var dataToSend = { signedTransaction: rawTx };
$.post('/v1/transaction/pushtx/', dataToSend, function (data) {}).fail( function() {} );
```
No return value in ``data``.

### Accept a trade offer:

#### Validate the "buyer" address - the address accepting the offer:
```
var dataToSend = { addr: buyer };
$.post('/v1/transaction/validateaddr/', dataToSend, function (data) {}).fail( function() {} );
```
``data`` will contain one of:

| Value                      | Meaning                  |
| -------------------------- | ------------------------ |
| ``{ "status": "OK" }``     | Address is a valid buyer |
| ``{ "status": "invalid pubkey" }``     | Public key on the blockchain for that address is invalid. |
| ``{ "status": "missing pubkey" }``     | No public key exists on the blockchain for this address.  Usually this means that the address has not yet sent any coins anywhere else. |
| ``{ "status": "invalid address" }``     | This address just isn't valid. |
#### Encode the acceptance:
```
var dataToSend = { buyer: buyer, pubKey: pubKey, amount: amount, tx_hash: tx_hash };
$.post('/v1/transaction/accept/', dataToSend, function (data) {

	//data should have fields sourceScript and transaction
	$('#sourceScript').val(data.sourceScript);
	$('#transactionBBE').val(data.transaction);

}).fail( function () {} );
```
#### Actually push up the signed transaction
```
var dataToSend = { signedTransaction: rawTx };
$.post('/v1/transaction/pushtx/', dataToSend, function (data) {}).fail( function() {} );
```
No return value in ``data``.
### Show Offers and Accepts by Address:
```
var postData = { 
	type: 'ADDRESS',
	address: address, 
	currencyType: ctype,   
	offerType: offer,      
};
$.post('/v1/exchange/offers/', postData , function(data,status,headers,config) { });
```

Where:

| Variable            | Possible Values              |
| ------------------- | ---------------------------- |
| address             | Bitcoin address |
| ctype        | 'tmsc' or 'msc'              |
| offerType      | 'SELL' (for Offers Made and Sold) or 'ACCEPT' (for Sales Accepted and Bought) | 

Resulting ``data``:
```
{
	"status": "OK",
	"data": {
		"offer_tx": [
			{
				"tx_hash": "d2caf8a19b29959e1fe34dc7a499bddb6ee40fd3fb3fa23e4cc00b827e11b818",
				"to_address": "unknown",
				"formatted_bitcoin_amount_desired": "0.01",
				"color": "bgc-new",
				"fee_required": "00c350",
				"tx_time": "1390480514000",
				"formatted_fee_required": "0.0005",
				"from_address": "17rExRiMaJGgHPVuYjHL21mEhfzbRPRkui",
				"index": "110",
				"tx_type_str": "Sell offer",
				"formatted_price_per_coin": "0.01",
				"currencyId": "00000002",
				"amount": "0000000005f5e100",
				"invalid": false,
				"details": "0.01",
				"bitcoin_amount_desired": "00000000000f4240",
				"method": "multisig",
				"dataSequenceNum": "01",
				"currency_str": "Test Mastercoin",
				"formatted_block_time_limit": "10",
				"amount_available": -52,
				"formatted_amount": "1.0",
				"icon": "selloffer",
				"formatted_amount_available": "-52.0",
				"tx_method_str": "multisig",
				"icon_text": "Sell Offer (2639 confirms)",
				"block_time_limit": "0a",
				"transactionType": "00000014",
				"baseCoin": "00",
				"block": "282035"
			}
		],
		"sold_tx": []
	}
}
```

### Get Detailed information about an offer
```
	var postData = { 
		type: 'TRANSACTIONBID',
		transaction: bidHash 
		currencyType: currencyType,   
		validityStatus: validity, 
	};
	$.post('/v1/exchange/offers/', postData , function(data,status,headers,config) {});
```

Where:

| Variable            | Possible Values              |
| ------------------- | ---------------------------- |
| bidHash             | Hash code of the transaction |
| currencytype        | 'tmsc' or 'msc'              |
| validityStatus      | 'ANY', 'VALID', 'EXPIRED', or 'INVALID' | 

Resulting ``data``:

```
{
	"status": "OK",
	"data": [
		{
			"tx_hash": "a24a6b5b38cec7047c14d2ce581c8576e233cf555bd318053d3bed12ebf803ce",
			"to_address": "1EdAjiApS5cCpHdH4RKPMab1xmMVRWjLvk",
			"from_address": "1F73UPD5xBKgTSRd8q6QhuncVmDnJAHxYV",
			"payment_expired": false,
			"color": "bgc-done",
			"update_fs": true,
			"tx_time": "1383470727000",
			"formatted_amount_accepted": 0.9,
			"sell_offer_txid": "e1a53bf47d64391294d07110f6cc9f94e56963e960829ad45886c9800047a6bf",
			"index": "28",
			"payment_txid": "Not available",
			"tx_type_str": "Sell accept",
			"formatted_price_per_coin": "0.000808",
			"currencyId": "00000002",
			"amount": "00000000055d4a80",
			"invalid": false,
			"details": "unknown_price",
			"formatted_amount_bought": "0.9",
			"formatted_amount_requested": "0.9",
			"method": "multisig",
			"dataSequenceNum": "01",
			"transactionType": "00000016",
			"status": "Closed",
			"btc_offer_txid": "8bb6b4ec970e9bbeb4500022b284767ce30f03dd0edf5d81b23f31dd6e26891d",
			"formatted_amount": "0.9",
			"payment_done": true,
			"bitcoin_required": "0.0007272",
			"icon": "sellaccept",
			"tx_method_str": "multisig",
			"icon_text": "Payment done",
			"currency_str": "Test Mastercoin",
			"baseCoin": "00",
			"block": "267672"
		},
		...
	]
}
```

### Get detailed transaction data
```
var postData = { 
	type: 'TRANSACTION',
	transaction: transaction, 
	currencyType: ctype   
};
$.post('/v1/exchange/offers/', postData , function(data,status,headers,config) {});
```

Where:

| Variable            | Possible Values              |
| ------------------- | ---------------------------- |
| transaction             | Bitcoin transaction hash |
| ctype        | 'tmsc' or 'msc'              |

Resulting ``data``:
```
{
	"status": "OK",
	"data": [
		{
			"tx_hash": "11020fe143c4a228bc093137a1cda8b7699c8ff9af78f7e4d31f3034f612f12c",
			"index": "343",
			"tx_type_str": "Simple send",
			"to_address": "1E68hpSZghk2UmFP3c3JCWnB9mpKWB1gyT",
			"from_address": "1HG3s4Ext3sTqBTHrgftyUzG3cvx5ZbPCj",
			"tx_method_str": "multisig",
			"color": "bgc-done",
			"tx_time": "1391662945000",
			"icon_text": "Simple send",
			"invalid": false,
			"currencyId": "00000002",
			"amount": "0000000000989680",
			"dataSequenceNum": "01",
			"currency_str": "Test Mastercoin",
			"formatted_amount": "0.1",
			"transactionType": "00000000",
			"icon": "simplesend",
			"baseCoin": "00",
			"method": "multisig",
			"block": "284397",
			"details": "1E68hpSZghk2UmFP3c3JCWnB9mpKWB1gyT"
		}
	]
}
```

### User Wallet Information

#### Wallet Data
User wallet data is signed and unsigned locally in the browser. The server only keeps a copy of the public address and encrypted password

Example

```
{
  "uuid": "02ddc252-7fb0-4e7d-c28e-be94a5dc56d0",
  "email": "user@email.com",
  "addresses": [
    {
      "address": "1JwSSubhmg6iPtRjtyqhUYYH7bZg3Lfy1T",
      "privkey": "6PRQ7ivF7LhQs7m4ZYbmj8Z1U7847LENYS22YBQNDLDXiVuKWZ8XDCEhjF"
    }
  ]
}
```

#### Syncing Wallet Information
```
var postData = {
  type: 'SYNCWALLET',
  wallet: walletData
};
$.post('/v1/user/wallet/sync/', postData, function (data) {}).fail( function() {} );
```

Where:

| Variable            | Possible Values              |
| ------------------- | ---------------------------- |
| type                | 'CREATEWALLET', 'SYNCWALLET' |
| wallet        | The wallet the user wants to save |

Returns status "EXISTS" if wallet already exists and type was "CREATEWALLET"

#### Restoring Wallet Information
```
var postData = {
  type: 'RESTOREWALLET',
  email: email
};
$.post('/v1/user/wallet/restore/', postData, function (data) {
  // Do something with data.wallet
})
```

Where:

| Variable            | Possible Values              |
| ------------------- | ---------------------------- |
| type                | 'RESTOREWALLET' |
| email | Email of wallet to retrieve |

Returns status 'OK' if wallet was found. Example:


```
{
  "status": "OK",
  "wallet":
  {
    "uuid": "02ddc252-7fb0-4e7d-c28e-be94a5dc56d0",
    "addresses": ["1JwSSubhmg6iPtRjtyqhUYYH7bZg3Lfy1T"],
    "email": "user@email.com",
    "addresses": [
      {
        "address": "1JwSSubhmg6iPtRjtyqhUYYH7bZg3Lfy1T",
        "privkey": "6PRQ7ivF7LhQs7m4ZYbmj8Z1U7847LENYS22YBQNDLDXiVuKWZ8XDCEhjF"
      }
    ]
  }
}
```

Returns status 'MISSING' if the wallet was not found.
