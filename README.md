# Kryptopal Alpha Client
API Docs available at http://api.testnet.kryptopalservice.com:8080/swagger-ui.html
https://documenter.getpostman.com/view/3526214/kryptopalapi/RW1Yofsg

## Getting started
This documentation is written for an Android project using Kotlin. The Kryptopal libraries will work with Java if you do not want to use Kotlin. The networking library used by this library is Retrofit2.

To begin using the Kryptopal you'll need to add the dependency to your gradle

Start by adding our repository to your project build.gradle
```
allprojects {  
    repositories {  
        google()  
        jcenter()  
        maven {  
            credentials {  
                username 'kryptopalreadonlyusers'  
  password 'eyJ2ZXIiOiIyIiwidHlwIjoiSldUIiwiYWxnIjoiUlMyNTYiLCJraWQiOiJTSC1CbmdqNXU4WWdXM0V0cVZmYjR0dms0SlUtblY5QTQzdExXYk1lTkEwIn0.eyJzdWIiOiJqZnJ0QDAxYzRiNDlwcWVtanZ3MTdna3Y2Z2MwNHQ2XC91c2Vyc1wva3J5cHRvcGFscmVhZG9ubHl1c2VycyIsInNjcCI6Im1lbWJlci1vZi1ncm91cHM6a3J5cHRvcGFsdGVzdHJlYWRlciBhcGk6KiIsImF1ZCI6ImpmcnRAMDFjNGI0OXBxZW1qdncxN2drdjZnYzA0dDYiLCJpc3MiOiJqZnJ0QDAxYzRiNDlwcWVtanZ3MTdna3Y2Z2MwNHQ2IiwiaWF0IjoxNTIzMjIzNTU1LCJqdGkiOiI2MjZmN2EwNC05NTQ1LTQwMmUtODQxZC1hNzYwZDU3MTJiYzMifQ.V02lskXhT2W1Fj3M1Ir1P69BZzZ1L3T2iiwnum4p7klqTmKPmpZihI8U5OUI3SCI1cKvWTFCDkAWZJtuC6CSd4EpjFH3SyurR22cSSaxjWvZO7AKuGeF3mi6db9HtVCbnz85-flguyM_9MljXDzQDKFszenNhRLHotuQPIB3zP3X6eta9C60YBGAiNMPklB-o9jfCE6IebGvIVM_Yn47Lhty1Blb2Cs2VstFbtr_s1kmItcA2sw864Q-Kk3a0EbEAu6fFR_kF3EZXi7-9lnzSOPCq2VIhncqUpooedjnCnzOigP2nhQ61_j703zqVkQa51MS5Cy9emHlDZiOk6G7LA'  
  }  
  
            url "https://kryptopal.jfrog.io/kryptopal/kryptopal-maven-public-repo/"  
  }  
    }  
}
```
Then add the following to your app dependencies. Replace `implementation` with `compile` if you are not building an android project.
```
implementation('com.kryptopal:kryptopal-java-client:1.0.0'){  
	exclude group: 'org.apache.oltu.oauth2'  //
}
implementation "org.threeten:threetenbp:1.3.6"	//needed for the client to work
implementation "io.gsonfire:gson-fire:1.8.2"	//needed for the client to work
```
You will also need to add in Retrofit2 and Gson or Moshi for the examples in these docs.
```
implementation 'com.squareup.retrofit2:retrofit:2.4.0'  
implementation 'com.squareup.retrofit2:converter-gson:2.4.0'  
implementation 'com.squareup.retrofit2:converter-scalars:2.4.0'  
implementation 'com.google.code.gson:gson:2.8.2'  
implementation 'com.squareup.moshi:moshi-kotlin:1.5.0'
```


## Creating a wallet
In order to sign transactions you'll need to use Geth or Web3J to create and manage your keystore. 

### Using [Geth](https://geth.ethereum.org/downloads/)
Details can be found [here](https://github.com/ethereum/go-ethereum/wiki/Mobile:-Account-management)
When using Geth your account is your wallet. To create an account on a mobile device you will probably want to use the light mode for faster account unlocking.
```
val keyStore = KeyStore("${context.filesDir}/keyStore", Geth.LightScryptN, Geth.LightScryptP)
val account = keyStore.newAccount("accountPassword")
```
To import an existing wallet from an outside source
```
val keyStore = KeyStore("${context.filesDir}/keyStore", Geth.LightScryptN, Geth.LightScryptP)
val account = keyStore.importECDSAKey("walletPrivateKey".hexToByteArray(), "walletPassword")
```

### Using [Web3J](https://github.com/web3j/web3j)
```
val fileName = WalletUtils.generateNewWalletFile("your password", File("/path/to/destination"))
val walletFile = File(fileName)
val credentials = WalletUtils.loadCredentials(password, walletFile)
```
to convert a Geth keystore's account into a web3J wallet
```
var walletUrl = gethAccount.url
walletUrl = walletUrl?.removePrefix("keystore://")
val walletFile = File(walletUrl)
val credentials = WalletUtils.loadCredentials(password, walletFile)
```



##  Send a transaction
To send a transaction using the Kryptopal api you need to

 - Send a request to `/v1/jsonrpc/sendKryptopalTransaction` specifying:
	 - From address
	 - To address
	 - Value (in Ether)
 - Sign the transaction specified in the response
 - Send the signed transaction through the network using `/v1/jsonrpc/sendSignedKryptopalTransactionUsingPOST`
 
 Let's go through this process one step at a time. Kryptopal's java client uses retrofit2 in order to make its network requests. 
```
val kpxService = ApiClient().createService(JsonrpcApi::class.java)  
val kryptopalTransaction = KryptopalTransaction()	// create the request's input parameter
kryptopalTransaction.from = fromAddress	// populate the fields
kryptopalTransaction.to = toAddress  
kryptopalTransaction.value = valueInEther.toString() // Convert BigDecimal to a string

val call = kpxService.getUnsignedKryptopalTransactionUsingPOST(kryptopalTransaction)  
call.enqueue(object: Callback<Any>{  
	override fun onFailure(call: Call<Any>?, t: Throwable?) {}
    override fun onResponse(call: Call<Any>?, response: Response<Any>?) {}  
})
```
`call.enqueue` will make an asynchronous request to our servers which will craft the necessary transaction object to sign. When the request completes we need to take the response and sign the transaction using Geth or Web3j. In order to manage the transaction received in the response we need to parse the response body. You can do this with Gson or Moshi.
```
override fun onResponse(call: Call<Any>?, response: Response<Any>?) {  
    if (response!!.isSuccessful) {  
        data class RawTransactionJson(val from: String, val to: String, val gas: String, val gasPrice: String, val value: String, val data: String, val nonce: String)  
        val jsonAdapter = Moshi.Builder().build().adapter(RawTransactionJson::class.java)  
        val unsignedTransactionData= jsonAdapter.fromJson(Gson().toJson(response.body()))  
  
        listener.onSuccess(jsonObject!!.from, jsonObject.to, jsonObject.gas, jsonObject.gasPrice, jsonObject.value, jsonObject.data, jsonObject.nonce)
        // better listener.onSuccess(unsignedTransactionData)
    } else {  
		// handle error scenario
    }  
}
```
Perform the transaction signing. See **Creating a Wallet** for details on how to set up a wallet.
```
val web3jTransaction = RawTransaction.createTransaction(  
	BigInteger(unsignedTransactionData.nonce.removePrefix("0x"), 16),  
	BigInteger(unsignedTransactionData.gasPrice.removePrefix("0x"), 16),  
	BigInteger(unsignedTransactionData.gas.removePrefix("0x"), 16),  
	unsignedTransactionData.to,  
	BigInteger(unsignedTransactionData.value.removePrefix("0x"), 16),  
	unsignedTransactionData.data  
)

val credentials = WalletUtils.loadCredentials(password, walletFile)
val signedByteArray = TransactionEncoder.signMessage(web3jTransaction , credentials)  
val signedTransaction = Numeric.toHexString(signedByteArray)
```
Now that we have signed the transaction the user is able post that transaction to the Kryptopal network.
```
val kpxService = kryptopalClient.createService(JsonrpcApi::class.java)  
val signedTransaction = SignedTransaction()  
signedTransaction.data = signedTransaction
val call = kpxService.sendSignedKryptopalTransactionUsingPOST(signedTransaction)
  
call.enqueue(object: Callback<Any> {  
    override fun onFailure(call: Call<Any>?, t: Throwable?) { }
    override fun onResponse(call: Call<Any>?, response: Response<Any>?) { }  
})
```
And that's all there is to it. `sendSignedKryptopalTransactionUsingPOST` will return the transaction receipt.
 

### Transaction Signing
#### Using Web3J

```
var walletUrl = walletWrapper.account?.url  
walletUrl = walletUrl?.removePrefix("keystore://")  
val walletFile = File(walletUrl)  
val credentials = WalletUtils.loadCredentials(password, walletFile)
val signedByteArray = TransactionEncoder.signMessage(transactionW3, credentials)  // Signing happens here
val txnData = Numeric.toHexString(signedByteArray)
```

### Checking the transaction receipt

##  RPC Methods

## Ticker Info
For your convenience Kryptopal provides the ability to query various sources for ticker information, however we currently we only support CoinMarketCap.

Using the `TickerSourceApi` we can easily fetch data about any coin. Here we request bitcoin using the `fetchTickerFromSourceWithIdUsingGet` endpoint.
``` 
val service = ApiClient().createService(TickerSourceApi::class.java)
val call = service.fetchTickerFromSourceWithIdUsingGET("coinmarketcap", "bitcoin", "")  
  
call.enqueue(object : Callback<Any> {  
    override fun onFailure(call: Call<Any>?, t: Throwable?) { }  
    override fun onResponse(call: Call<Any>?, response: Response<Any>?) {  
        Timber.d("withID: ${response?.body().toString()}")  
    }  
})
```
```
[
  {
    "id": "bitcoin",
    "name": "Bitcoin",
    "symbol": "BTC",
    "rank": "1",
    "price_usd": "9204.13",
    "price_btc": "1.0",
    "market_cap_usd": "156480214889",
    "available_supply": "17001087.0",
    "total_supply": "17001087.0",
    "percent_change_1h": "-0.24",
    "percent_change_24h": "3.53",
    "percent_change_7d": "10.63",
    "last_updated": "1524803975",
    "24h_volume_usd": "8621840000.0"
  }
]
```


You can use `fetchTickerFromSourceUsingGET` to pass in parameters such as conversion and start. This allows you to fetch multiple tickers at once. See the API docs for more information on parameters. In this example we fetch the ticker with the third highest market cap and request the value in Euros.
```
val service = ApiClient().createService(TickerSourceApi::class.java)    
val params = mutableListOf("limit=1")  
params.add("convert=EUR")  
params.add("start=2")  
val call = service.fetchTickerFromSourceUsingGET("coinmarketcap", params)  
  
call.enqueue(object : Callback<Any> {  
    override fun onFailure(call: Call<Any>?, t: Throwable?) { }  
  
    override fun onResponse(call: Call<Any>?, response: Response<Any>?) {  
        Timber.d("onResponse: ${response?.body().toString()}")  
    }  
})
```
The request will return JSON containing information about the ticker
```
[
  {
    "id": "ripple",
    "name": "Ripple",
    "symbol": "XRP",
    "rank": "3",
    "price_usd": "0.833506",
    "price_btc": "0.00009067",
    "market_cap_usd": "32628595409.0",
    "available_supply": "39146203398.0",
    "total_supply": "99992334929.0",
    "percent_change_1h": "-0.94",
    "percent_change_24h": "3.02",
    "percent_change_7d": "0.24",
    "last_updated": "1524802441",
    "price_eur": "0.6885809778",
    "market_cap_eur": "26955331011.0",
    "24h_volume_usd": "934322000.0",
    "24h_volume_eur": "771867696.572"
  }
]
```
