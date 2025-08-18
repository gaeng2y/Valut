```shell
--header 'Content-Type: multipart/form-data; boundary=--------------------------938871668557517864980248' \
--form 'data=@"/Users/shkim/Pictures/ship-8308680_640.jpg"' \
--form 'content="123"'
```
의 형태로 호출해야된다.

예제로는

```swift
private extension HTTPClient {
    func createBody(boundary: String, data: Data, mimeType: String, filename: String) -> Data {
        let body = NSMutableData()
        let boundaryPrefix = "--\(boundary)\r\n"

        body.append(boundaryPrefix)
        body.append("Content-Disposition: form-data; name=\"file\"; filename=\"\(filename)\"\r\n")
        body.append("Content-Type: \(mimeType)\r\n\r\n")
        body.append(data)
        body.append("\r\n")
        body.append("--".appending(boundary.appending("--")))

        return body as Data
    }
}
```

처럼 호출할 수 있다.