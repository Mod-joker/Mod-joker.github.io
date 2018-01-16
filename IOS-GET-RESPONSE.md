# iOS Swift and AFNetworking: Get response data and status code in case of failure
If you’re using the AFNetworking library to perform HTTP queries in an iOS or OS X application, either with Objective-C or Swift, there’s no simple way to get the response content in case of failure.

Let’s assume the following POST query that send and receive JSON:
```
let manager = AFHTTPSessionManager(baseURL: NSURL("http://frog-api.localhost"))
manager.requestSerializer = AFJSONRequestSerializer()
manager.responseSerializer = AFJSONResponseSerializer()
let params = [
    "foo": "bar"
]

manager.POST("/app_dev.php/oauth/v2/token", parameters: params, success: {
    (task: NSURLSessionDataTask!, responseObject: AnyObject!) in
        println("success")

}, failure: {
    (task: NSURLSessionDataTask!, error: NSError!) in
        println("error")
})
```
We’ll now talk about the failure listener, and how to get the response status code and then the data.

## Get the status code
The NSURLSessionDataTask object as a response attribute which is a NSHTTPURLResponse object. The status code, which is stored in the statusCode attribute, can simply be accessible this way:
```
let response = task.response as NSHTTPURLResponse
println(response.statusCode)
```
## Get the response content
Getting the response content is a little bit more complicated: we have to create our own response serializer. In this case, we’ll override the responseObjectForResponse method of the AFJSONResponseSerializer.

The following implementation simply add the result data in the userInfo of the error object which is accessible in the failure closure:
```
class JSONResponseSerializer: AFJSONResponseSerializer
{
    override func responseObjectForResponse (response: NSURLResponse, data data: NSData, error error: AutoreleasingUnsafePointer) -> AnyObject
    {
        var json = super.responseObjectForResponse(response, data: data, error: error) as AnyObject

        if error.memory? {
            var errorValue = error.memory!
            var userInfo = errorValue.userInfo as NSDictionary
            var copy = userInfo.mutableCopy() as NSMutableDictionary

            copy["data"] = json

            error.memory = NSError(domain: errorValue.domain, code: errorValue.code, userInfo: copy)
        }

        return json
    }
}
```
Then, you’ve to change the response serializer you want to use with your AFNetworking manager:
```
manager.responseSerializer = JSONResponseSerializer()
```
Based on that we can access to the response content using the error’s userInfo:
```
if let data = error.userInfo["data"] as? NSDictionary {
    println(data)
}
```
> 转载自https://sroze.io/ios-swift-and-afnetworking-get-response-data-and-status-code-in-case-of-failure-69cee3c33eb1
