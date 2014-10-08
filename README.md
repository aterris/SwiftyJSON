#SwiftyJSON

SwiftyJSON makes it easy to deal with JSON data in Swift.

1. [Why is the typical JSON handling in Swift NOT good](#Why-is-the-typical-JSON-handling-in-Swift-NOT-good)
1. [Requirements](#requirements)
1. [Integration](#integration)
1. [Usage](#usage)
	1. [Initialization](#initialization)
	1. [Subscript](#subscript)
	1. [Error](#Error)
	1. [Loop](#loop)
	1. [Optional getter](#optional-getter)
	1. [Non-optional getter](#non-optional-getter)
	1. [Setter](#setter)
	1. [Raw object](#raw-object)
	1. [Literal convertible](#literal-convertible)
1. [Work with Alamofire](#work-with-alamofire)
	
##Why is the typical JSON handling in Swift NOT good?
Swift is very strict about types, it's good while explicit typing left us little chance to make mistakes. 
But while dealing with things that naturally implicit about types such as JSON, it's painful.

Take the Twitter API for example: say we want to retrieve a user's "name" value of some tweet in Swift (according to Twitter's API https://dev.twitter.com/docs/api/1.1/get/statuses/home_timeline)

```JSON

[
  {
    ......
    "text": "just another test",
    ......
    "user": {
      "name": "OAuth Dancer",
      "favourites_count": 7,
      "entities": {
        "url": {
          "urls": [
            {
              "expanded_url": null,
              "url": "http://bit.ly/oauth-dancer",
              "indices": [
                0,
                26
              ],
              "display_url": null
            }
          ]
        }
      ......
    },
    "in_reply_to_screen_name": null,
  },
  ......]
  
```

The code would look like this:

```swift

let jsonObject : AnyObject! = NSJSONSerialization.JSONObjectWithData(dataFromTwitter, options: NSJSONReadingOptions.MutableContainers, error: nil)
if let statusesArray = jsonObject as? NSArray{
    if let aStatus = statusesArray[0] as? NSDictionary{
        if let user = aStatus["user"] as? NSDictionary{
            if let userName = user["name"] as? NSDictionary{
                //Finally We Got The Name
                
            }
        }
    }
}

```

It's not good.

Even if we use optional chaining, it would also cause a mess:

```swift

let jsonObject : AnyObject! = NSJSONSerialization.JSONObjectWithData(dataFromTwitter, options: NSJSONReadingOptions.MutableContainers, error: nil)
if let userName = (((jsonObject as? NSArray)?[0] as? NSDictionary)?["user"] as? NSDictionary)?["name"]{
  //What A disaster above
}

```
An unreadable mess for something like this should really be simple!

With SwiftyJSON all you have to do is:

```swift

let json = JSON(data: dataFromNetworking)
if let userName = json[0]["user"]["name"].string{
  //Now you got your value
}

```

And don't worry about the Optional Wrapping thing, it's done for you automatically

```swift

let json = JSON(data: dataFromNetworking)
if let userName = json[999999]["wrong_key"]["wrong_name"].string{
    //Calm down, take it easy, the ".string" property still produces the correct Optional String type with safety
} else {
    //Print the error
    println(json[999999]["wrong_key"]["wrong_name"])
}

```

## Requirements

- iOS 7.0+ / Mac OS X 10.9+
- Xcode 6.0

##Integration

CocoaPods is not fully supported for Swift yet, to use this library in your project you should:  

1. for Projects just drag SwiftyJSON.swift to the project tree
2. for Workspaces you may include the whole SwiftyJSON.xcodeproj as suggested by @garnett

## Usage

####Initialization
```swift
let json = JSON(data: dataFromNetworking)
```
```swift
let json = JSON(jsonObject)
```

####Subscript
```swift
let name = json[0]["name"].stringValue
```

####Loop
```swift
let json = JSON(data:dataFromNetworking)
	//If json is .Dictionary
for (key: String, subJson: JSON) in json {
	//Do something you want
}

//If json is .Array
//The `index` is 0..<json.count's string value
for (index: String, subJson: JSON) in json {
//Do something you want
}
```
####Error
Use subscript to get/set value in Array or Dicitonary

*  If json is an array, the app may crash with index may be out  of bounds.
*  If json is a dictionary, it will get `nil` without the reason. 
*  If json is not an array or a dictionary, the app may crash with wrong selector exception.

It will never happen in SwiftyJSON

```swift
let json = JSON(["name", "age"])
let name = json[999].string {
	//Do something you want
} else {
	println(json[999].error) // "Array[999] is out of bounds"
}
```
```swift
let json = JSON(["name":"Jack", "age": 25])
let name = json["address"].string {
	//Do something you want
} else {
	println(json["address"].error) // "Dictionary["address"] does not exist"
}
```
```swift
let json = JSON(12345)
let age = json[0].string {
	//Do something you want
} else {
	println(json[0])       // "Array[0] failure, It is not an array"
	println(json[0].error) // "Array[0] failure, It is not an array"
}

let name = json["name"].string {
	//Do something you want
} else {
	println(json["name"])       // "Dictionary[\"name"] failure, It is not an dictionary"
	println(json["name"].error) // "Dictionary[\"name"] failure, It is not an dictionary"
}
```
```swift
json["name"] = JSON("new-name")
json["0"] = JSON("new-name")
```

####Optional getter
```swift
//NSNumber
if let id = json["user"]["favourites_count"].number {
   //Do something you want
} else {
   //Print the error
   println(json["user"]["favourites_count"].error)
}
```
```swift
//String
if let id = json["user"]["name"].string {
   //Do something you want
} else {
   //Print the error
   println(json["user"]["name"])
}
```
```swift
//Bool
if let id = json["user"]["is_translator"].bool {
   //Do something you want
} else {
   //Print the error
   println(json["user"]["is_translator"])
}
```
```swift
//Int
if let id = json["user"]["id"].int {
   //Do something you want
} else {
   //Print the error
   println(json["user"]["id"])
}
...
```
####Non-optional getter
Non-optional getter is named `xxxValue`
```swift
//If not a Number or nil, return 0
let id: Int = json["id"].intValue
```
```swift
//If not a String or nil, return ""
let name: String = json["name"].stringValue
```
```swift
//If not a Array or nil, return []
let list: Array<JSON> = json["list"].arrayValue
```
```swift
//If not a Dictionary or nil, return [:]
let user: Dictionary<String, JSON> = json["user"].dictionaryValue
```

####Setter
```swift
json["id"].int =  1234567890
json["coordinate"].double =  8766.766
json["name"].string =  "Jack"
json.array = [1,2,3,4]
json.dictionary = ["name":"Jack", "age":25]
```
####Raw object
```swift
let jsonObject: AnyObject = json.object
```
```swift
if let jsonObject: AnyObject = json.toRaw
```
```swift
if let json = JSON.fromRaw(object) {
	//object can be converted to JSON
} else {
	//object can not be converted to JSON
}
```
####Literal convertible
More info about the literal convertible. please read [Swift Literal Convertibles](http://nshipster.com/swift-literal-convertible/) by Mattt Thompson
```swift
//StringLiteralConvertible
let json:JSON = "I'm a json"
```
```swift
//IntegerLiteralConvertible
let json:JSON =  12345
```
```swift
//BooleanLiteralConvertible
let json:JSON =  true
```
```swift
//FloatLiteralConvertible
let json:JSON =  2.8765
```
```swift
//DictionaryLiteralConvertible
let json:JSON =  ["I":"am", "a":"json"]
```
```swift
//ArrayLiteralConvertible
let json:JSON =  ["I", "am", "a", "json"]
```
```swift
//NilLiteralConvertible
let json:JSON =  nil
```
```swift
//With subscript in array
var json:JSON =  [1,2,3]
json[0] = 100
json[1] = 200
json[2] = 300
json[999] = 300 //Don't worry, nothing will happen
```
```swift
//With subscript in dictionary
var json:JSON =  ["name":"Jack", "age": 25]
json["name"] = "Mike"
json["age"] = "25" //It's OK to set String
json["address"] = "L.A." // Add the "address": "L.A." in json
```

##Work with Alamofire

To use Alamofire and SwiftyJSON, try [Alamofire-SwiftyJSON](https://github.com/SwiftyJSON/Alamofire-SwiftyJSON).
