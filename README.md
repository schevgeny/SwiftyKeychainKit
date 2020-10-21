# SwiftyKeychainKit

![Platforms](https://img.shields.io/badge/platforms-ios%20-lightgrey.svg)
[![Build Status](https://travis-ci.org/andriyslyusar/SwiftyKeychainKit.svg?branch=master)](https://travis-ci.org/andriyslyusar/SwiftyKeychainKit)
![Swift version](https://img.shields.io/badge/swift-5.0%20%7C%205.1-orange.svg)
[![CocoaPods compatible](https://img.shields.io/badge/CocoaPods-compatible-4BC51D.svg?style=flat)](#cocoapods)
[![SPM compatible](https://img.shields.io/badge/SPM-compatible-4BC51D.svg?style=flat)](#swift-package-manager)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](#carthage)

SwiftyKeychainKit is a simple Swift wrapper for Keychain Services API with the benefits of static typing. Define your keys in one place, use value types easily, and get extra safety and convenient compile-time checks for free.

## Features
* Static typing and compile-time checks
* Support Gereric and Internet passwords
* Throwing and Result type get methods
* Easy way to implement support for custom types

## Usage

### Basic

```swift
let keychain = Keychain(service: "com.swifty.keychain")
let accessTokenKey = KeychainKey<String>(key: "accessToken")

// set or modify value
try? keychain.set("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9", for : accessTokenKey)

// Get value 
let value = try keychain.get(accessTokenKey)

// Remove value 
try keychain.remove(accessTokenKey)

// Remove all values 
try keychain.removeAll()
```

### Instantiation

```swift
// Generic password
let keychain = Keychain(service: "com.swifty.keychainkit")

// Internet password
let keychain = Keychain(server: URL(string: "https://www.google.com")!, protocolType: .https)
```

### Define keys 

For extra convenience, define your keys by extending `KeychainKeys` class and adding static properties:

```swift
extension KeychainKeys {
    static let username = KeychainKey<String>(key: "username")
    static let age = KeychainKey<Int>(key: "age")
}
```

and later in the code use shortcut dot syntax:

```swift
// 

try? keychain.set("John Snow", for: .username)

// get
let username = try keychain.get(.username)
```

### Geting values

You can use `subscripts` and `dynamicCallable` syntax sugar to get value as `Result<ValueType, KeychainError>`

```swift
let username = try keychain[.username].get()

// or 

if case .success(let age) = keychain[.age] {
    ...
}
```

```swift
let username = try keychain(.username).get()

// or 

if case .success(let age) = keychain(.age) {
    ...
}
```

Both `subscripts` and `dynamicCallable` syntaxt available only for geting values. Currently Swift language limitation do not allow implement setter with error handling.

### Default values

You can provide default value for get method and it will used than **keychain key is nil** and **no error throw**.

```swift
try keychain.get(.username, default: "Daenerys Targaryen")

// or

try keychain[.age, default: 18].get() 
```

## Supported types
- Int
- String
- Double
- Float
- Bool
- Data

### Codable
```swift
struct MyStruct: Codable, KeychainSerializable { ... }
```

### NSCoding
```swift
class MyClass: NSObject, NSCoding, KeychainSerializable { ... }
```

### Custom types
In order to set/get your own custom type that we don't support, you need to confirm it `KeychainSerializable` and implement `KeychainBridge` for this type.

As an example saving `Array<String>` using `JSONSerialization`:

```swift 
extension Array: KeychainSerializable where Element == String  {
    public static var bridge: KeychainBridge<[String]> { return KeychainBridgeStringArray() }
}
```

```swift
class KeychainBridgeStringArray: KeychainBridge<[String]> {
    public override func set(_ value: [String], forKey key: String, in keychain: Keychain) throws {
        guard let data = try? JSONSerialization.data(withJSONObject: value, options: []) else {
            fatalError()
        }
        try? persist(value: data, key: key, keychain: keychain)
    }

    public override func get(key: String, from keychain: Keychain) throws -> [String]? {
        guard let data = try? find(key: key, keychain: keychain) else {
            return nil
        }

        return (try? JSONSerialization.jsonObject(with: data, options: [])) as? [String]
    }
}
```

## Supported attributes
**Common**
- [x] kSecAttrAccessGroup 
- [x] kSecAttrAccessible 
- [x] kSecAttrDescription
- [x] kSecAttrComment
- [ ] kSecAttrCreator
- [ ] kSecAttrType
- [x] kSecAttrLabel
- [x] kSecAttrIsInvisible
- [x] kSecAttrIsNegative
- [x] kSecAttrAccount
- [x] kSecAttrSynchronizable

**Generic password**
- [ ] kSecAttrAccessControl
- [x] kSecAttrService 
- [ ] kSecAttrGeneric

**Internet password**
- [x] kSecAttrSecurityDomain
- [x] kSecAttrServer
- [x] kSecAttrProtocol
- [x] kSecAttrAuthenticationType
- [x] kSecAttrPort
- [x] kSecAttrPath


## Requirement

**Swift** version **5.0**

Platform     | Availability
------------ | -------------
iOS          | >= 8.0
macOS        | -
tvOS         | - 
watchOS      | -

## Installation
#### CocoaPods
```ruby
pod 'SwiftyKeychainKit', '1.0.0-beta.2'
```

#### Swift Package Manager
```swift
let package = Package(
    dependencies: [
        .Package(url: "https://github.com/andriyslyusar/SwiftyKeychainKit.git", .exact("1.0.0-beta.2"))
    ]
)
```

#### Carthage
```ruby
github "andriyslyusar/SwiftyKeychainKit" "1.0.0-beta.2"
```

## Acknowledgement 
* [SwiftyUserDefaults](https://github.com/sunshinejr/SwiftyUserDefaults) - API design inspiration
* [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess) - API wrapper knowledge source

## Author
Andriy Slyusar  
* https://twitter.com/andriyslyusar
* https://github.com/andriyslyusar 

## License

SwiftyKeychainKit is available under the MIT license. See the [LICENSE file](https://github.com/andriyslyusar/SwiftyKeychainKit/blob/master/LICENSE) for more info.
