## 1. 导入依赖

```xml
<!-- https://mvnrepository.com/artifact/com.konghq/unirest-java -->
<dependency>
    <groupId>com.konghq</groupId>
    <artifactId>unirest-java</artifactId>
    <version>3.14.5</version>
</dependency>
```

## 2. 配置 `application.yml`

```yml
# SMMS对象存储
smms:
    token: xxxxxxxxxxxxxxxxxxxxx
```

## 3. 写入`SmmsVO.java`

```java
import lombok.Data;

@Data
public class SmmsVO {
    private Boolean success;
    private String url;
    private String hash;
    private String message;
}
```

## 4. 写入`SmmsManager.java`

```java
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import com.wsj.springbootinit.model.vo.SmmsVO;
import kong.unirest.HttpResponse;
import kong.unirest.Unirest;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.io.File;
import java.lang.reflect.Type;
import java.util.HashMap;
import java.util.Map;

@Component
public class SmmsManager {

    public static final String url = "https://smms.app/api/v2";
    @Value("${smms.token}")
    private String token;

    public SmmsVO uploadImg(File file) {
        return doUpload(file);
    }

    public SmmsVO uploadImg(String filePath) {
        File file = new File(filePath);
        return doUpload(file);
    }

    public SmmsVO deleteImg(String imgHash) {
        // 使用unirest
        HttpResponse<String> response = Unirest.get(url + "/delete/" + imgHash)
                .header("Authorization", token)
                .asString();
        // Json转map
        Map<String, Object> mapSource = convertJsonToMap(response.getBody());
        SmmsVO smmsVO = new SmmsVO();
        smmsVO.setSuccess((Boolean) mapSource.get("success"));
        smmsVO.setMessage((String) mapSource.get("message"));
        return smmsVO;
    }

    private SmmsVO doUpload(File file) {
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
        SmmsVO smmsVO = new SmmsVO();
        if ((Boolean) mapSource.get("success")) {
            Map<String, Object> mapData = (Map<String, Object>) mapSource.get("data");
            smmsVO.setSuccess(true);
            smmsVO.setUrl((String) mapData.get("url"));
            smmsVO.setHash((String) mapData.get("hash"));
            return smmsVO;
        } else {
            smmsVO.setSuccess(false);
            smmsVO.setUrl((String) mapSource.get("images"));
            smmsVO.setMessage((String) mapSource.get("code"));
            return smmsVO;
        }
    }

    private Map<String, Object> convertJsonToMap(String json) {
        // 使用TypeToken来保留Map<String, Object>的类型信息
        Type type = new TypeToken<Map<String, Object>>() {
        }.getType();
        // 将JSON字符串转换为Map
        return new Gson().fromJson(json, type);
    }
}
```

## 5. 示例

上传

```java
SmmsVO res = SmmsManager.uploadImg("C:\\Users\\86178\\Desktop\\file\\win11壁纸\\61 (小).jpg");

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
SmmsVO res2 = SmmsManager.deleteImg("YCADqXkxUsvBw5mhFQLWNMgfIH");

{success=true, message=File delete success.}
{success=false, message=File already deleted.}
```

使用案例

```java
@PostMapping("/userAvatar")
public BaseResponse<SmmsVO> uploadUserAvatar(@RequestPart("file") File file) {
    // 先删除图床中原有的图片再上传
    if (smmsManager.deleteImg("here_is_imgHash").getSuccess()) {
        SmmsVO smmsVO = smmsManager.uploadImg(file);
        //String url = smmsVO.getUrl();
        //String imgHash = smmsVO.getHash();
        return ResultUtils.success(smmsVO);
    } else {
        throw new BusinessException(ErrorCode.OPERATION_ERROR, "更新头像失败");
    }
}
```

