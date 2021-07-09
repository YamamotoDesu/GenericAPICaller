# GenericAPICaller-Swift
https://youtu.be/Ot_fk9LZxEk

# ツール、開発環境など
- Xcode Version 12.4 (12D4e)
- Swift 5、Swift

# 完成イメージ
![Simulator Screen Shot - iPhone 12 - 2021-07-10 at 00 46 25](https://user-images.githubusercontent.com/47273077/125104468-5ff09880-e118-11eb-8255-fdc8aaff6b82.png)


# Free fake API for testing 
https://jsonplaceholder.typicode.com/users
<pre>
[
  {
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "suite": "Apt. 556",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": {
        "lat": "-37.3159",
        "lng": "81.1496"
      }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
      "name": "Romaguera-Crona",
      "catchPhrase": "Multi-layered client-server neural-net",
      "bs": "harness real-time e-markets"
    }
  },
  .....中略.......
      "phone": "024-648-3804",
    "website": "ambrose.net",
    "company": {
      "name": "Hoeger LLC",
      "catchPhrase": "Centralized empowering task-force",
      "bs": "target end-to-end models"
    }
  }
]
</pre>
https://jsonplaceholder.typicode.com/todos
<pre>
[
  {
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
  },
  {
    "userId": 1,
    "id": 2,
    "title": "quis ut nam facilis et officia qui",
    "completed": false
  },
  .....中略.......
    {
    "userId": 10,
    "id": 200,
    "title": "ipsam aperiam voluptates qui",
    "completed": false
  }
]
</pre>

# ソースコードの説明
## Generic API
<pre>
extension URLSession {
    enum CustomError: Error {
        case invalidUrl
        case invalidData
    }
    
    func request<T: Codable>(
        url:URL?,
        expecting: T.Type,
        completion: @escaping (Result<T, Error>) -> Void
    ) {
        guard let url = url else {
            completion(.failure(CustomError.invalidUrl))
            return
        }
        
        let task = dataTask(with: url) { data, _, error in
            guard let data = data else {
                if let error = error {
                    completion(.failure(error))
                } else {
                    completion(.failure(CustomError.invalidData))
                }
                return
            }
            
            do {
                let result = try JSONDecoder().decode(expecting, from: data)
                completion(.success(result))
            }
            catch {
                completion(.failure(error))
            }
        }
        task.resume()
    }
}

</pre>

## Models
<pre>
struct User: Codable {
    let name: String
    let email: String
}

struct ToDoListItem: Codable {
    let title: String
    let completed: Bool
}
</pre>

## Call API
    // User
    func fetch() {
        URLSession.shared.request(
            url: Constants.usersUrl,
            expecting: [User].self
        ) { [weak self] result in
            switch result {
            case .success(let users):
                DispatchQueue.main.async {
                    self?.models = users
                    self?.table.reloadData()
                }
            case .failure(let error):
                print(error)
            }
        }
    }
    
    // ToDoList
    func fetchItems() {
        URLSession.shared.request(
            url: Constants.todolistUrl,
            expecting: [ToDoListItem].self
        ) { [weak self] result in
            switch result {
            case .success(let items):
                DispatchQueue.main.async {
                    self?.models = items
                    self?.table.reloadData()
                }
            case .failure(let error):
                print(error)
            }
        }
    }
</pre>

