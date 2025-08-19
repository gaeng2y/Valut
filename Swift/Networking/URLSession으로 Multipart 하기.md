### curl 요청 기준 Request
```shell
curl --location --request POST 'https://api.goforawalk.site/api/v1/footsteps' \
--header 'Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxIiwiaXNzIjoiYXBpLmdvZm9yYXdhbGsuc2l0ZSIsIm5pY2tuYW1lIjoidGVzdCIsImlhdCI6MTc1MTc5NzA4OSwiZXhwIjo1MzUxNzk3MDg5fQ.jqYqOww4qFX1c69xnLjBQBcvXOqYVcPB4z8K1UEn_F16lPajtoK2Sv-TQpWOM7XRwP4GKiNnRQYuKKYVh8ODqQ' \
--header 'Accept: */*' \
--header 'Host: api.goforawalk.site' \
--header 'Connection: keep-alive' \
--header 'Content-Type: multipart/form-data; boundary=--------------------------648743068807640196135474' \
--form 'data=@"/Users/shkim/Pictures/ship-8308680_640.jpg"' \
--form 'content="123"'
```
의 형태로 호출해야된다.

Swift에 맞게 적용하면

```swift
private static func createMultipartBody(
        with requestDTO: CreateFootstepRequestDTO,
        boundary: String
    ) -> Data {
        var body = Data()
        
        func appendString(_ string: String) {
            if let data = string.data(using: .utf8) {
                body.append(data)
            }
        }
        
        // --- Part 1: 이미지 데이터 (data) ---
        // cURL의 --form 'data=@"/path/to/image.jpg"' 부분
        appendString("--\(boundary)\r\n")
        // Content-Disposition 헤더에 name과 함께 filename을 명시해야 합니다.
        appendString("Content-Disposition: form-data; name=\"data\"; filename=\"\(requestDTO.fileName)\"\r\n")
        // 해당 파트의 Content-Type을 명시해줍니다.
        appendString("Content-Type: \(requestDTO.mimeType)\r\n\r\n")
        body.append(requestDTO.data)
        appendString("\r\n")
        
        // --- Part 2: 텍스트 데이터 (content) ---
        // cURL의 --form 'content="123"' 부분
        appendString("--\(boundary)\r\n")
        appendString("Content-Disposition: form-data; name=\"content\"\r\n\r\n")
        appendString(requestDTO.content)
        appendString("\r\n")
        
        // --- Body 종료 ---
        appendString("--\(boundary)--\r\n")
        
        return body
    }
 public func makeMultiPartURLRequest(isBypass: Bool) throws -> URLRequest {
        guard var urlComponent = try makeURLComponents(isBypass: isBypass) else {
            throw NetworkError.urlRequestError(.urlComponentError)
        }
        
        if let queryItems = try getQueryParameters() {
            urlComponent.queryItems = queryItems
        }
        
        guard let url = urlComponent.url else {
            throw NetworkError.invalidURL
        }
        
        var urlRequest = URLRequest(url: url)
        
        if let httpBody = try getBodyParameters() {
            urlRequest.httpBody = httpBody
        }
        
        if let headers {
            headers.forEach { key, value in
                urlRequest.setValue(value, forHTTPHeaderField: key)
            }
        }
        
        // multipart 요청에서는 Content-Type을 덮어쓰지 않음 (헤더에서 이미 설정됨)
        urlRequest.httpMethod = httpMethod.rawValue
        
        return urlRequest
    }
```

위와 같이 적용할 수 있다.