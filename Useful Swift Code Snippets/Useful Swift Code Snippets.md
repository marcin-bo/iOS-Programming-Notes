# Useful Swift Code Snippets

# Table of Contents

1. [Networking](#networking)
    1. [Create `URLRequest` Instance](#networking_urlrequest)
        1. [From String](#urlrequest_1)
        1. [From `URLComponents`](#urlrequest_2)
    1. [Configure a `URLRequest` Instance](#urlrequest_configure)
        1. [Set Custom Headers For `URLRequest` Instance](#urlrequest_custom_headers)
        1. [Set HTTP Method](#urlrequest_http_method)
        1. [Set JSON Body For POST HTTP Request](#urlrequest_body)
            1. [Using `JSONSerialization`](#urlrequest_body_jsonserialization)
            1. [Using `Encodable`](#urlrequest_body_encodable)
    1. [Send a URL Request](#send_urlrequest)
        1. [Using `URLSession.dataTask`](#send_urlrequest_1)
        1. [Using Swift Concurrency](#send_urlrequest_2)
        1. [Using Combine](#send_urlrequest_3)
    1. [Example of Handling URL Response](#url_response_handling)
    1. [Download an Image From URL](#download_image)
        1. [Using `Data(contentsOf:)` and `UIImage(data:)`](#download_image_1)
        1. [Using `URLSession.dataTask` and `UIImage(data:)`](#download_image_2)  
1. [JSON Handling](#json_handling)
    1. [Serializing Dictionary to `Data`](#json_serialize)
    1. [Deserializing `Data` to Dictionary](#json_deserialize)
    1. [JSON Encoding](#json_encode)
    1. [JSON Decoding](#json_decode)
    1. [Custom JSON Encoding And Decoding](#custom_json)


# Networking <a name="networking"></a>

## Create `URLRequest` Instance <a name="networking_urlrequest"></a>

### From String <a name="urlrequest_1"></a>

```swift
let url = "https://www.example.com/path"
var request = URLRequest(url: url)
```

### From `URLComponents` <a name="urlrequest_2"></a>

```swift
var components = URLComponents()
components.scheme = "https"
components.host = "api.github.com"
components.path = "/search/repositories"
components.queryItems = [
    URLQueryItem(name: "q", value: "iOS"),
    URLQueryItem(name: "sort", value: "updated")
]

let request: URLRequest? = components.url
```

## Configure a `URLRequest` Instance <a name="urlrequest_configure"></a>

### Set Custom Headers For `URLRequest` Instance <a name="urlrequest_custom_headers"></a>

```swift
var request = URLRequest(url: "https://www.example.com/path")

// Set HTTP headers
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
request.setValue("application/json", forHTTPHeaderField: "Accept")
request.setValue("someAuthToken", forHTTPHeaderField: "Authorization")
```

### Set HTTP Method <a name="urlrequest_http_method"></a>

```swift
var request = URLRequest(url: "https://www.example.com/path")
request.httpMethod = "POST"
```

### Set JSON Body For POST HTTP Request <a name="urlrequest_body"></a>

#### Using `JSONSerialization` <a name="urlrequest_body_jsonserialization"></a>

```swift
var request = URLRequest(url: "https://www.example.com/path")
request.httpMethod = "POST"

let body = ["name": "iPhone"]
let bodyData = try? JSONSerialization.data(withJSONObject: body, options: [])
request.httpBody = bodyData
```

#### Using `Encodable` <a name="urlrequest_body_encodable"></a>

```swift
struct Product: Encodable {
    let name: String
}

var request = URLRequest(url: "https://www.example.com/path")
request.httpMethod = "POST"

let product = Product(name: 123)

do {
    let bodyData = try JSONEncoder().encode(message)
    request.httpBody = bodyData
} catch {
    // Handle encoding error
}
```

## Send a URL Request <a name="send_urlrequest"></a>

### Using `URLSession.dataTask` <a name="send_urlrequest_1"></a>

```swift
// 1. Create URLRequest
let request = URLRequest(url: "https://www.example.com/path")

// 2. Create URLSessionDataTask
let dataTask = URLSession.shared.dataTask(with: request) { data, response, error in
    if let data = data {
        
        // Parse JSON data
        // Catch any parsing error
        // Switch to main thread if needed for UI updates
        
    } else if let error = error {
        print("Error sending request: \(error)")
    } else {
        print("Unknown error sending request")
    }
}

// 3. Start URLSessionDataTask
dataTask.resume()

// 4. [Optional] You can cancel the request if needed at any time
dataTask.cancel()
```

### Using Swift Concurrency <a name="send_urlrequest_2"></a>

```swift
// 1. Create Task
let task: Task<Void, Never> = Task {
    do {
        // 2. Create URLRequest
        let request = URLRequest(url: "https://www.example.com/path")
        
        let (data, _) = try await URLSession.shared.data(for: request)
        
        // Parse JSON data
        // Catch any parsing error
        // Switch to main thread if needed for UI updates
        
    } catch {
        print("Error sending request: \(error)")
    }
}

// 3. [Optional] You can cancel the request if needed at any time
task.cancel()
```

### Using Combine <a name="send_urlrequest_3"></a>

```swift
// 1. Create URLRequest
let request = URLRequest(url: "https://www.example.com/path")

// 2. Create Publisher
let cancellable = URLSession.shared.dataTaskPublisher(for: request)
    .sink(receiveCompletion: { completion in
        if case .failure(let error) = completion {
            print("Error sending request: \(error)")
        }
    }, receiveValue: { data, response in
        // Parse JSON data
        // Catch any parsing error
        // Switch to main thread if needed for UI updates
    })
    
// 3. [Optional] You can cancel the request if needed at any time
cancellable.cancel()
```

## Example of Handling URL Response <a name="url_response_handling"></a>

```swift
let dataTask = session.dataTask(with: request) { data, response, error in
        
    // Handle error    
    if let error = error {
        print("Request Error: \(error)")
        return
    }
    
    // Handle valid response
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        print("Invalid Response received from the server")
        return
    }
    
    // Extract response data
    guard let responseData = data else {
        print("Empty data received from the server")
        return
    }
    
    // Decode the response
    do {
        let product = try JSONDecoder().decode(Product.self, from: data)
        print(product)
        
        // Switch to the main thread if needed
        
    } catch {
        print(error)
    }
}

dataTask.resume()
```


## Download an Image From URL <a name="download_image"></a>

### Using `Data(contentsOf:)` and `UIImage(data:)` <a name="download_image_1"></a>

```swift
DispatchQueue.global().async { // Use [weak self] if needed
    if let data = try? Data(contentsOf: url) {
        if let image = UIImage(data: data) {
            
            // Switch to the Main Thread
            DispatchQueue.main.async {
                // Use image instance
            }
        }
    }
}
```

### Using `URLSession.dataTask` and `UIImage(data:)` <a name="download_image_2"></a>

```swift
// 1. Create URL
let url = URL(string: "https://example.com/image.png")!

// 2. Create URLSessionDataTask
let dataTask = URLSession.shared.dataTask(with: url) { (data, _, error) in // Use [weak self] if needed
    if let data = data {
        // Switch to the Main Thread and update UI
        DispatchQueue.main.async {
            let image = UIImage(data: data)
        }
    } else if let error = error {
        print("Error sending request: \(error)")
    } else {
        print("Unknown error sending request")
    }
}

// 3. Start URLSessionDataTask
dataTask.resume()

// 4. [Optional] You can cancel the request if needed at any time
dataTask.cancel()
```

# JSON Handling <a name="json_handling"></a>

## Serializing Dictionary to `Data` <a name="json_serialize"></a>

```swift
let jsonObject = ["name": "iPhone"]
let data = try? JSONSerialization.data(withJSONObject: jsonObject, options: [])
```

## Deserializing `Data` to Dictionary <a name="json_deserialize"></a>

```swift
let jsonResponse = try? JSONSerialization.jsonObject(with: responseData, options: .mutableContainers) as? [String: Any]
print(jsonResponse)
```

## JSON Encoding <a name="json_encode"></a>

```swift
struct Product: Encodable {
    let name: String
    let price: Int
    
    enum CodingKeys: String, CodingKey {
        case name = "product_name" // Only name will be included
    }
}

let product = Person(price: "iPhone", price: 999)
do {
    let data = try JSONEncoder().encode(product)
    print(String(data: data, encoding: .utf8)!) // {"product_name": "iPhone"}
} catch {
    print(error)
}
```

## JSON Decoding <a name="json_decode"></a>

```swift
struct Product: Decodable {
    let name: String
    let price: Int
    
    enum CodingKeys: String, CodingKey {
        case name = "product_name" // Only name will be included
    }
}

let data = "{\"product_name\": \"iPhone\"}".data(using: .utf8)!
do {
    let product = try JSONDecoder().decode(Product.self, from: data)
    print(product) // Product(name: "iPhone", price: 0)
} catch {
    print(error)
}
```

## Custom JSON Encoding And Decoding <a name="custom_json"></a>

```swift
struct Product: Codable {
    let name: String
    let price: Int
    
    enum CodingKeys: String, CodingKey {
        case name
        case price
    }
    
    func encode(to encoder: Encoder) throws {
        // Validate name
        guard !name.isEmpty else {
            throw EncodingError.invalidValue(
                name, 
                EncodingError.Context(codingPath: [CodingKeys.name], debugDescription: "Name cannot be empty")
            )
        }
        
        // Validate price
        guard price > 0 else {
            throw EncodingError.invalidValue(
                price, 
                EncodingError.Context(codingPath: [CodingKeys.price], debugDescription: "Price cannot be less than 0")
            )
        }
        
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(name, forKey: .name)
        try container.encode(price, forKey: .price)
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        // Name decoding
        let name = try container.decode(String.self, forKey: .name)
        
        // Validate name
        guard !name.isEmpty else {
            throw DecodingError.valueNotFound(String.self, DecodingError.Context(codingPath: [CodingKeys.name], debugDescription: "Name cannot be empty"))
        }
        
        // Price decoding
        let price = try container.decode(Int.self, forKey: .price)
        
        // Validate price
        guard price > 0 else {
            throw DecodingError.valueNotFound(
                Int.self, 
                DecodingError.Context(codingPath: [CodingKeys.price], debugDescription: "Price cannot be less than 0")
            )
        }
        
        // Initialize, values are correct
        self.init(name: name, price: price)
    }
}
```
