# URLSessionDemo
Basics of swift ios URLSession GET POST request . First we will check the basic and simple way to doing get post request using URLSession. Then we will check much cleaner way for callign api request in real life application

### Tech Stack
1. swift 5
2. URLSession
3. JsonSerialization
4. Encodable / Decodable
5. URLComponents
6. Generics
7. Closures

### Simple get request using urlsession

```swift
func getData() {
    let url = URL(string: "PASTE YOUR GET URL HERE")
    
    URLSession.shared.dataTask(with: url!) { (data, response, error) in
        
        if error == nil {
            
            if let data = data {
                
                do {
                    if let jsonData = try JSONSerialization.jsonObject(with: data, options: []) as? Dictionary<String, Any> {
                        print(jsonData["NAME_OF_THE_KEY"])
                    }
            
                } catch let err {
                    print(err)
                }
            }
            
        }
    }
}
```



### Simple post request
```swift
func postData() {
    
    let url = URL(string: "PASTE YOUR POST URL HERE")
    
    var urlRequest = URLRequest(url: url!)
    
    let body = ["key": "value", "key": true, "key": 1] as [String : Any]
    
    do {
        urlRequest.httpBody = try JSONSerialization.data(withJSONObject: body, options: [])
    } catch let err {
        print("err \(err)")
    }
    
    urlRequest.addValue("application/json", forHTTPHeaderField: "content-type")
    
    urlRequest.httpMethod = "post"
    
    URLSession.shared.dataTask(with: urlRequest) { (data, response, error) in
        
    }
    
    
}
```


### cleaner way for networking using Encodable & Decodable and Generics & Closures


#### our sample respone

```swift
struct SampleResponse: Decodable
{
    let userName, email, phone: String
    
    enum CodingKeys: String, CodingKey {
        case userName = "user_name"
        case email
        case phone
    }
}

struct SamplePostBody: Encodable {
    let username, password: String
    
    enum CodingKeys: String, CodingKey {
        case username = "username"
        case password = "password"
    }
}
```



> Creating a APIService which will be used for calling api using urlsession. We can call get post request using this simple service from anywhere
```swift
struct APIService {
    
    func requestService<T:Decodable>(urlRequest: URLRequest, resultType: T.Type, completionHandler: @escaping (_ result: T?,_ error: Error?) -> Void)  {
        
        URLSession.shared.dataTask(with: urlRequest) { (data, response, error) in
            
            if error == nil {
                
                if let data = data {
                    
                    do {
                        let jsonData = try JSONDecoder().decode(T.self, from: data)
                        completionHandler(jsonData, nil)
                    } catch let err {
                        print(err)
                        completionHandler(nil, err)
                    }
                    
                }
                
            }else {
                completionHandler(nil, error)
            }
            
        }
        
    }
}
```


> In the ViewModel class we can create fucntion for our get post request

```swift
class ViewModel {
    
    var apiService: APIService2
    
    
    init(_apiService: APIService2) {
        apiService = _apiService
    }
    
    func getData() {
        
        let url = URL(string: "GET_URL")
        
        var urlRequest = URLRequest(url: url!)
        
        urlRequest.httpMethod = "get"
        
        apiService.requestService(urlRequest: urlRequest, resultType: SampleResponse.self) { (response, error) in
            
            
            if error == nil {
                
                print(response)
                
            }
            
        }
    }
    
    
    func postData() {
        
        let url = URL(string: "POST_URL")
        
        var urlRequest = URLRequest(url: url!)
        
        urlRequest.httpMethod = "post"
        
        let body = SamplePostBody(username: "username", password: "password")
        
        do {
            
            let postBody = try JSONEncoder().encode(body)
            
            urlRequest.httpBody = postBody
            
            urlRequest.addValue("application/json", forHTTPHeaderField: "content-type")
            
            urlRequest.addValue("Authorization", forHTTPHeaderField: "TOKEN")
            
        } catch let err {
            print(err)
        }
        
        apiService.requestService(urlRequest: urlRequest, resultType: SampleResponse.self) { (response, error) in
            
        }
    }
}
```


### Passing query parameters

```swift
func queryParameterInUrlExample(color: String, model: String) {
    
    let url = URL(string: "https://dummyurl.com/api/details?color=\(color)&model=\(model)")
    
    
    // insted of passing in url we can use this
    
    let urlWithooutQueryParameter = URL(string: "https://dummyurl.com/api/details")
    
    var queryParameterURl = URLComponents(url: urlWithooutQueryParameter!, resolvingAgainstBaseURL: false)
    
    
    queryParameterURl?.queryItems = [
        URLQueryItem(name: "color", value: color),
        URLQueryItem(name: "model", value: model)
    ]
    
    var urlRequest = URLRequest(url: (queryParameterURl?.url)!)
}
```

### Pass query parameter dynamically

```swift
struct QueryPatameterItem: Encodable {
    let color, model: String?
}


extension Encodable {
    
    func convertToQueryParameter() -> [URLQueryItem]? {
        
        do {
            
            let encode = try JSONEncoder().encode(self)
            
            let jsonDictionary = (try? JSONSerialization.jsonObject(with: encode, options: [])).flatMap{
                $0 as? [String: Any?]
            }
            
            if jsonDictionary != nil {
                
                var queryItems = [URLQueryItem]()
                
                jsonDictionary?.forEach({ (key, value) in
                    if value != nil {
                        if let strValue = value as? String  {
                            queryItems.append(URLQueryItem(name: key, value: strValue))
                        }
                    }
                })
                
                return queryItems
            }
            
        } catch let err {
            print(err)
        }
        
        return nil
    }
}


func queryParameterAdvanced(color: String, model: String) {
    
    
    let urlWithooutQueryParameter = URL(string: "https://dummyurl.com/api/details")
    
    var queryParameterURl = URLComponents(url: urlWithooutQueryParameter!, resolvingAgainstBaseURL: false)
    
    
    let queryitems = QueryPatameterItem(color: "color", model: nil)
    
    queryParameterURl?.queryItems = queryitems.convertToQueryParameter()
    
    var urlRequest = URLRequest(url: (queryParameterURl?.url)!)
    
}
```



### Tutorial link 
1. [Code Cat Youtube Play list](https://www.youtube.com/watch?v=-7EyKipJltc&list=PLb5R4QC2DtFuXr4177KQ2lIXOkqwq97a4&index=1)

