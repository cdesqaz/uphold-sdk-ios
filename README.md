# Uphold SDK for iOS [![Build Status](https://travis-ci.com/uphold/uphold-sdk-ios.svg?token=p3QCGNiSpUZ3XCyWxkTB&branch=master)](https://travis-ci.com/uphold/uphold-sdk-ios) [![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

Uphold is a next generation platform that allows anyone to transfer and exchange value for free, instantly and securely.

The Uphold SDK for iOS provides an easy way for developers to integrate iOS applications with the [Uphold API](https://uphold.com/en/developer/api).

## Requirements

    * Xcode 7
    * Swift 2
    * Carthage or CocoaPods

## Installation

### Using [CocoaPods](https://cocoapods.org)

1. Create `Podfile`.

    ```
    platform :ios, '9.0'
    use_frameworks!

    pod 'UpholdSdk'
    ```

2. Run `pod install`.

### Using [Carthage](https://github.com/Carthage/Carthage)

1. Create `Cartfile`.

    ```
    github "uphold/uphold-sdk-ios" ~> 0.1.0
    ```

2. Run `carthage update --platform iOS`.

## Basic usage

In order to learn more about the Uphold API, please visit the [developer website](https://uphold.com/en/developer).

To use the SDK you must first register an Application and obtain a unique `client_id` and `client_secret` combination. We recommend your first app be [registered in the Sandbox environment](https://sandbox.uphold.com/dashboard/profile/applications/developer/new), so you can safely play around during development.

From the application page in your account you can get the `Client ID`, `Client Secret` , configure the `redirect URI` and the desired `Scopes`.

### Authenticate User

In order to allow users to be re-directed back to the application after the authorization process, you’ll need to associate your custom `scheme` with your app by adding the following keys into the [`Info.plist`](https://github.com/uphold/uphold-sdk-ios/blob/master/Example/Info.plist) file:

* CFBundleURLTypes - The list of URLs types to be handled by the application.
    * CFBundleURLSchemes - The custom application schemes.

For instance, our demo application has the following configuration:

```xml
<key>CFBundleURLTypes</key>
	<array>
		<dict>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>uphold-demo</string>
			</array>
		</dict>
	</array>
```

We start the authentication process by instantiating the UpholdClient and then calling the `beginAuthorization` method:

```swift
/// LoginViewController.swift

let upholdClient = UpholdClient()
let authorizationViewController = upholdClient.beginAuthorization(self, clientId: CLIENT_ID, scopes: scopes, state: state)
```

In the `AppDelegate` class you'll need to implement the method `application(application: UIApplication, openURL url: NSURL, options: [String : AnyObject]) -> Bool` that is called when the user completes the authorization process.

```swift
/// AppDelegate.swift

func application(application: UIApplication, openURL url: NSURL, options: [String : AnyObject]) -> Bool {
    loginViewController.completeAuthorization(url)

    return true
}
```

To complete the authorization process you'll need to call the `completeAuthorization` method from the `upholdClient` and get the user bearer token from the authentication response.

```swift
/// LoginViewController.swift

upholdClient.completeAuthorization(authorizationViewController, clientId: `CLIENT_ID`, clientSecret: `CLIENT_SECRET`, grantType: "authorization_code", state: state, uri: url).then { (response: AuthenticationResponse) -> () in
    // Get the user bearer token from the authenticationResponse.
}
```

To get the current user information, just instantiate the Uphold client with the user bearer token and then call the `getUser()` function:

```swift
let upholdClient = UpholdClient(bearertoken: bearerToken)
upholdClient.getUser().then { (user: User) -> () in
    /// The user information is available at the user object.
}
```

### Get user cards with chaining

```swift
let upholdClient = UpholdClient(bearerToken: bearerToken)
upholdClient.getUser().then { (user: User) -> Promise<[Card]> in
    return user.getCards()
.then { (cards: [Card]) -> () in
    /// Do something with the list of cards.         
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.
}
```

### Get user cards

```swift
user.getCards().then { (cards: [Card]) -> () in
    /// Do something with the list of cards.        
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.            
}
```

### Get ticker

```swift
/// Instantiate the client. In this case, we don't need an
/// AUTHORIZATION_TOKEN because the Ticker endpoint is public.
let upholdClient = UpholdClient()

/// Get tickers.
upholdClient..getTickers().then { (rateList: [Rate]) -> () in
    /// Do something with the rates list.
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.
}
```

Or you could get a ticker for a specific currency:

```swift
/// Get tickers for BTC.
upholdClient.getTickersByCurrency("BTC").then { (rateList: [Rate]) -> () in
    /// Do something with the rates list.
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.
}
```

### Create and commit a new transaction

```swift
let transactionDenominationRequest = TransactionDenominationRequest(amount: "1.0", currency: "BTC")
let transactionRequest = TransactionRequest(denomination: transactionDenominationRequest, destination: "foo@bar.com")

card.createTransaction(transactionRequest).then { (transaction: Transaction) -> () in
    /// Commit the transaction.
    transaction.commit(TransactionCommitRequest("Commit message"))
}.error({ (error: ErrorType) -> Void in
    /// Do something with the error.            
})
```

If you want to commit the transaction on the creation process, call the `createTransaction` method with the second parameter set to `true`.

```swift
card.createTransaction(true, transactionRequest: transactionRequest)
```

### Get all public transactions

```swift
/// Instantiate the client. In this case, we don't need an
/// AUTHORIZATION_TOKEN because the Ticker endpoint is public.
let upholdClient = UpholdClient()

let paginator: Paginator<Transaction> = client.getReserve().getTransactions()

/// Get the list of transactions.
paginator.elements.then { (transactions: [Transaction]) -> () in
    /// Do something with the list of transactions.            
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.         
}

/// Get the next page of transactions.
paginator.getNext().then { (transactions: [Transaction]) -> () in
    /// Do something with the list of transactions.         
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.         
}
```

Or you could get a specific public transaction:

```swift
/// Get one public transaction.
upholdClient.getReserve().getTransactionById("a97bb994-6e24-4a89-b653-e0a6d0bcf634").then { (transaction: Transaction) -> () in
    /// Do something with the list of transactions.     
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.      
}
```

### Get reserve status

```swift
/// Instantiate the client. In this case, we don't need an
/// AUTHORIZATION_TOKEN because the Ticker endpoint is public.
let upholdClient = UpholdClient()

/// Get the reserve summary of all the obligations and assets within it.
client.getReserve().getStatistics().then { (reserveStatistics: [ReserveStatistics]) -> () in
    /// Do something with the reserve statistics.    
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.     
}
```

### Pagination
Some endpoints will return a paginator. Here are some examples on how to handle it:

```swift
/// Get public transactions paginator.
let paginator: Paginator<Transaction> = client.getReserve().getTransactions()

/// Get the first page of transactions.
paginator.elements.then { (transactions: [Transaction]) -> () in
    /// Do something with the list of transactions.            
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.         
}

/// Check if the paginator has a valid next page.
paginator.hasNext().then { (hasNext: Bool) -> () in
    /// Do something with the hasNext.     
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.         
}

/// Get the number of paginator elements.
paginator.count().then { (count: Int) -> () in
    /// Do something with the count.    
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.     
}

/// Get the next page.
paginator.getNext().then { (transactions: [Transaction]) -> () in
    /// Do something with the list of transactions.         
}.error { (error: ErrorType) -> Void in
    /// Do something with the error.         
}
```

## Uphold SDK sample

Check the [sample application](https://github.com/uphold/uphold-sdk-ios/tree/master/Example) to explore an application using the Uphold iOS SDK.
