# Wallet Usage

### Configure Networking and Pair clients

Make sure what you properly configure Networking and Pair Clients first 
- [Networking](../core/networking-configuration.md)
- [Pairing](../core/pairing-usage.md)

### Instantiate a client

Create an AppMetadata object. It will describe your application and define its appearance in a web browser.
Then configure the `Auth` instance Account object.

```swift
Auth.configure(account: Account("eip155:56:0xe5EeF1368781911d265fDB6946613dA61915a501")!)
```

### Subscribe for Auth publishers
When your `Auth` instance receives requests or responses from a peer client, it will publish a related event. So you should set a subscription to handle them.

```swift
Auth.instance.authRequestPublisher
    .receive(on: DispatchQueue.main)
    .sink { [unowned self] _ in
        //handle event
    }.store(in: &publishers)
```
Following publishers are available to subscribe:

```swift
public var authRequestPublisher: AnyPublisher<AuthRequest, Never> {
public var authResponsePublisher: AnyPublisher<(id: RPCID, result: Result<Cacao, AuthError>), Never> {
public let socketConnectionStatusPublisher: AnyPublisher<SocketConnectionStatus, Never>

```

### Connect Clients

Your Wallet should allow users to scan a QR code generated by dapps and handle universal linking. However, you are responsible for implementing it on your own. Once you derive a URI from the QR code call pair method:

```swift
try await Auth.instance.pair(uri: uri)
```

### Handle requests from a dapp

After pairing with dapp, your wallet will be subscribing for authentication requests. Requests will be published by `authRequestPublisher`. When a wallet receives a request, you want to present it to the user and request a signature. After the user signs the authentication message, the wallet should respond to a dapp.

`Type` parameter represent signature validation method which will be used on DApp side. Supported signature validation methods: [EIP191](https://eips.ethereum.org/EIPS/eip-191), [EIP1271](https://eips.ethereum.org/EIPS/eip-1271). In both cases message will be signed with [EIP191](https://eips.ethereum.org/EIPS/eip-191) standard.

```swift
let signer = MessageSignerFactory.create()
let signature = try signer.sign(message: request.message, privateKey: privateKey, type: .eip191)
try await Auth.instance.respond(requestId: request.id, signature: signature)
```

In case user rejects an authentication request, call:
```swift
try await Auth.instance.reject(requestId: request.id)
```

### Get pending requests

if you've missed some requests you can querry them with
```swift 
Auth.instance.getPendingRequests()
```


### Where to go from here
- Try our Showcase wallet that is part of WalletConnectSwiftV2 repository.
- Build API documentation in XCode: go to Product -> Build Documentation

