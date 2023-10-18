# 1. 配置 `application.yml`

```yml
# SMMS对象存储
oss:
  smms:
    url: https://smms.app/api/v2
    token: xxxxxxxxxxxxxxxxxxxxx
```

# 2. 写入`OssSmmsManager.java`

```java
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import kong.unirest.HttpResponse;
import kong.unirest.Unirest;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

import java.io.File;
import java.lang.reflect.Type;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class OssSmmsManager {

    @Value("${oss.smms.url}")
    private String url;
    @Value("${oss.smms.token}")
    private String token;

    public Map<String, Object> uploadImg(File file) {
        return doUpload(file);
    }

    public Map<String, Object> uploadImg(String filePath) {
        File file = new File(filePath);
        return doUpload(file);
    }

    public Map<String, Object> deleteImg(String imgHash) {
        // 使用unirest
        HttpResponse<String> response = Unirest.get(url + "/delete/" + imgHash)
                .header("Authorization", token)
                .asString();
        return convertJsonToMap(response.getBody());
    }

    private Map<String, Object> doUpload(File file) {
        // 构造 Header
        Map<String, String> headerMap = new HashMap<>();
        headerMap.put("Authorization", token);
        // 构造 params
        Map<String, Object> paramsMap = new HashMap<>();
        paramsMap.put("smfile", file);
        // 发送
        HttpResponse<String> response = Unirest.post(url + "/upload")
                .header("Authorization", token)
                .field("smfile", file)
                .asString();
        // Json转map
        Map<String, Object> mapSource = convertJsonToMap(response.getBody());
        // 根据是否上传成功来构造返回值
        Map<String, Object> result = new HashMap<>();
        if ((Boolean) mapSource.get("success")) {
            Map<String, Object> mapData = (Map<String, Object>) mapSource.get("data");
            result.put("success", true);
            result.put("url", mapData.get("url"));
            result.put("hash", mapData.get("hash"));
            return result;
        } else {
            result.put("success", false);
            result.put("url", mapSource.get("images"));
            result.put("message", mapSource.get("code"));
            return result;
        }
    }

    private Map<String, Object> convertJsonToMap(String json) {
        // 使用TypeToken来保留Map<String, Object>的类型信息
        Type type = new TypeToken<Map<String, Object>>() {}.getType();
        // 将JSON字符串转换为Map
        return new Gson().fromJson(json, type);
    }
}
```

# 3. 示例

上传

```java
Map<String, Object> res = ossSmmsManager.uploadImg("C:\\Users\\86178\\Desktop\\file\\win11壁纸\\61 (小).jpg");

// 返回结果 - 成功
{
    success=true, 
    url=https://s2.loli.net/2023/10/18/eBFjomuhCzA3OQJ.jpg, 
    hash=YCADqXkxUsvBw5mhFQLWNMgfIH
}
// 失败
{
    success=false, 
    message=image_repeated, 
    url=https://s2.loli.net/2023/10/18/WC8HmP2rVk5xNL3.jpg
}
```

删除

```java
Map<String, Object> res2 = ossSmmsManager.deleteImg("YCADqXkxUsvBw5mhFQLWNMgfIH");

{success=true, message=File delete success.}
{success=false, message=File already deleted.}
```

使用案例

```java
Map<String, Object> resultMap;
try {
    // 上传
    resultMap = ossSmmsManager.uploadImg(file);
    // 删除原来的图片
    UserVO userVO = userService.getLoginUser(request);
    ossSmmsManager.deleteImg(userVO.getAvatarHash());
} catch (Exception e) {
    throw new BusinessException(ErrorCode.OPERATION_ERROR, "头像更新失败");
}
```

