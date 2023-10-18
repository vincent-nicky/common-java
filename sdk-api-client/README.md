# 1. 配置`application.yml`

```yml
wsj:
  api:
    client:
      access-key: xxxxxxxxxxxxxxxxxxxxxxx
      secret-key: xxxxxxxxxxxxxxxxxxxxxxx
```

# 2. 调用

```java
//构建请求参数
ApiDataFieldRequest apiDataFieldRequest = new ApiDataFieldRequest();
apiDataFieldRequest.setPath(url);
apiDataFieldRequest.setMethod(method);
// 注意 requestHeaders 和 requestParams 的格式：{"aa": "11","bb": "22"}
apiDataFieldRequest.setHeadersJson(requestHeaders);
apiDataFieldRequest.setParamsJson(requestParams);

Map<String,Object> resultJson = myApiClient.invokeInterface(apiDataFieldRequest);
```

或者

```java
//构建请求参数
UserVO loginUser = userService.getLoginUser(request);
String accessKey = loginUser.getAccessKey();
String secretKey = loginUser.getSecretKey();
ApiDataFieldRequest apiDataFieldRequest = new ApiDataFieldRequest();
apiDataFieldRequest.setPath(url);
apiDataFieldRequest.setMethod(method);
// 注意 requestHeaders 和 requestParams 的格式：{"aa": "11","bb": "22"}
apiDataFieldRequest.setHeadersJson(requestHeaders);
apiDataFieldRequest.setParamsJson(requestParams);

Map<String,Object> resultJson = myApiClient.invokeInterface(accessKey,secretKey, apiDataFieldRequest);
```

# 3. 使用案例

```java
// begin 构建请求参数
UserVO loginUser = userService.getLoginUser(request);
String accessKey = loginUser.getAccessKey();
String secretKey = loginUser.getSecretKey();
ApiDataFieldRequest apiDataFieldRequest = new ApiDataFieldRequest();
apiDataFieldRequest.setPath(interfaceInfo.getUrl());
apiDataFieldRequest.setMethod(interfaceInfo.getMethod());
// 处理请求头的格式
List<InvokeRequest.Field> headersList = invokeRequest.getRequestHeaders();
String requestHeaders = "{}";
if (headersList != null && headersList.size() > 0) {
    JsonObject jsonObject = new JsonObject();
    for (InvokeRequest.Field field : headersList) {
        jsonObject.addProperty(field.getFieldName(), field.getValue());
    }
    requestHeaders = gson.toJson(jsonObject);
}
apiDataFieldRequest.setHeadersJson(requestHeaders);
// 处理参数的格式
List<InvokeRequest.Field> paramsList = invokeRequest.getRequestParams();
String requestParams = "{}";
if (paramsList != null && paramsList.size() > 0) {
    JsonObject jsonObject = new JsonObject();
    for (InvokeRequest.Field field : paramsList) {
        jsonObject.addProperty(field.getFieldName(), field.getValue());
    }
    requestParams = gson.toJson(jsonObject);
}
apiDataFieldRequest.setParamsJson(requestParams);
// end
Map<String,Object> resultJson = myApiClient.invokeInterface(accessKey,secretKey, apiDataFieldRequest);
```

