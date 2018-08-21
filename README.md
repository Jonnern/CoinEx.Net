# ![Icon](https://github.com/JKorf/CoinEx.Net/blob/master/Resources/icon.png?raw=true) CoinEx.Net 

![Build status](https://travis-ci.org/JKorf/CoinEx.Net.svg?branch=master)

A .Net wrapper for the CoinEX API as described on [CoinEx](https://github.com/coinexcom/coinex_exchange_api/wiki), including all features the API provides using clear and readable objects.

**If you think something is broken, something is missing or have any questions, please open an [Issue](https://github.com/JKorf/CoinEx.Net/issues)**

---
Also check out my other exchange API wrappers:
<table>
<tr>
<td><a href="https://github.com/JKorf/Bittrex.Net"><img src="https://github.com/JKorf/Bittrex.Net/blob/master/Resources/icon.png?raw=true"></a>
<br />
<a href="https://github.com/JKorf/Bittrex.Net">Bittrex</a>
</td>
<td><a href="https://github.com/JKorf/Bitfinex.Net"><img src="https://github.com/JKorf/Bitfinex.Net/blob/master/Resources/icon.png?raw=true"></a>
<br />
<a href="https://github.com/JKorf/Bitfinex.Net">Bitfinex</a>
</td>
<td><a href="https://github.com/JKorf/Binance.Net"><img src="https://github.com/JKorf/Binance.Net/blob/master/Resources/binance-coin.png?raw=true"></a>
<br />
<a href="https://github.com/JKorf/Binance.Net">Binance</a>
</td>
</table>

And other API wrappers based on CryptoExchange.Net:
<table>
<tr>
<td><a href="https://github.com/Zaliro/Switcheo.Net"><img src="https://github.com/Zaliro/Switcheo.Net/blob/master/Resources/switcheo-coin.png?raw=true"></a>
<br />
<a href="https://github.com/Zaliro/Switcheo.Net">Switcheo</a>
</td>
</tr>
</table>


## Donations
Donations are greatly appreciated and a motivation to keep improving.

**Btc**:  12KwZk3r2Y3JZ2uMULcjqqBvXmpDwjhhQS  
**Eth**:  0x069176ca1a4b1d6e0b7901a6bc0dbf3bb0bf5cc2  
**Nano**: xrb_1ocs3hbp561ef76eoctjwg85w5ugr8wgimkj8mfhoyqbx4s1pbc74zggw7gs  


## Installation
![Nuget version](https://img.shields.io/nuget/v/coinex.net.svg)  ![Nuget downloads](https://img.shields.io/nuget/dt/CoinEx.Net.svg)
Available on [Nuget](https://www.nuget.org/packages/CoinEx.Net/).
```
pm> Install-Package CoinEx.Net
```
To get started with CoinEx.Net first you will need to get the library itself. The easiest way to do this is to install the package into your project using  [NuGet](https://www.nuget.org/packages/CoinEx.Net/). Using Visual Studio this can be done in two ways.

### Using the package manager
In Visual Studio right click on your solution and select 'Manage NuGet Packages for solution...'. A screen will appear which initially shows the currently installed packages. In the top bit select 'Browse'. This will let you download net package from the NuGet server. In the search box type 'CoinEx.Net' and hit enter. The CoinEx.Net package should come up in the results. After selecting the package you can then on the right hand side select in which projects in your solution the package should install. After you've selected all project you wish to install and use CoinEx.Net in hit 'Install' and the package will be downloaded and added to you projects.

### Using the package manager console
In Visual Studio in the top menu select 'Tools' -> 'NuGet Package Manager' -> 'Package Manager Console'. This should open up a command line interface. On top of the interface there is a dropdown menu where you can select the Default Project. This is the project that CoinEx.Net will be installed in. After selecting the correct project type  `Install-Package CoinEx.Net`  in the command line interface. This should install the latest version of the package in your project.

After doing either of above steps you should now be ready to actually start using CoinEx.Net.
## Getting started
After installing it's time to actually use it. To get started you have to add the CoinEx.Net namespace:  `using CoinEx.Net;`.

CoinEx.Net provides two clients to interact with the CoinEx API. The  `CoinExClient`  provides all rest API calls. The  `CoinExSocketClient`  provides functions to interact with the websocket provided by the CoinEx API. Both clients are disposable and as such can be used in a  `using`statement.

Most API methods are available in two flavors, sync and async:
````C#
public void NonAsyncMethod()
{
    using(var client = new CoinExClient())
    {
        var result = client.GetMarketList();
    }
}

public async Task AsyncMethod()
{
    using(var client = new CoinExClient())
    {
        var result2 = await client.GetMarketListAsync();
    }
}
````

## Examples
Examples can be found in the Examples folder.


## Response handling
All API requests will respond with an CallResult object. This object contains whether the call was successful, the data returned from the call and an error if the call wasn't successful. As such, one should always check the Success flag when processing a response.
For example:
```C#
using(var client = new CoinExClient())
{
	var result = client.GetMarketList();
	if (result.Success)
		Console.WriteLine($"# markets: {result.Data.Length}");
	else
		Console.WriteLine($"Error: {result.Error.Message}");
}
```
## Options & Authentication
The default behavior of the clients can be changed by providing options to the constructor, or using the `SetDefaultOptions` before creating a new client. Api credentials can be provided in the options.

## Websockets
The CoinEx.Net socket client provides several socket endpoint to which can be subscribed and follow this function structure

```C#
var client = new CoinExSocketClient();

var subscribeResult = client.SubscribeToMarketState("ETHBTC", (market, stateData) =>
{
	// handle data
});


```

**Handling socket events**

Subscribing to a socket stream returns a CoinExStream object. This object can be used to be notified when a socket closes or an error occures:
````C#
var sub = client.SubscribeToMarketStateUpdates("ETHBTC", data =>
{
	Console.WriteLine("Received state update");
});

sub.Data.Closed += () =>
{
	Console.WriteLine("Socket closed");
};

sub.Data.Error += (e) =>
{
	Console.WriteLine("Socket error " + e.Message);
};
````

**Unsubscribing from socket endpoints:**

Sockets streams can be unsubscribed by using the `client.UnsubscribeFromStream` method in combination with the stream subscription received from subscribing:
```C#
var client = new CoinExSocketClient();

var successSub = client.SubscribeToMarketStateUpdates("ETHBTC", (data) =>
{
	// handle data
});

client.UnsubscribeFromStream(successSub.Data);

```

Additionaly, all sockets can be closed with the `UnsubscribeAllStreams` method. Beware that when a client is disposed the sockets are automatically disposed. This means that if the code is no longer in the using statement the eventhandler won't fire anymore. 

## Release notes
* Version 0.0.6 - 20 aug 2018
	* Fix for default api credentials getting disposed

* Version 0.0.5 - 20 aug 2018
	* Updated CryptoExchange.Net for bug fix

* Version 0.0.4 - 17 aug 2018
	* Added handling for incosistent data in socket update
	* Added additional logging
	* Small reconnection fixes

* Version 0.0.3 - 16 aug 2018
	* Added client interfaces
	* Fixed minor Resharper warnings

* Version 0.0.2 - 13 aug 2018
	* Upped CryptoExchange.Net to fix bug

* Version 0.0.1 - 13 aug 2018
	* Initial release