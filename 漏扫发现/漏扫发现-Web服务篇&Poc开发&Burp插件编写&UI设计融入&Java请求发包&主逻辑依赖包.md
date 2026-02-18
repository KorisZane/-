https://gitee.com/stemmm/burp-api-drops
https://mp.weixin.qq.com/s/5JqM0G6Uxc2HRI5EfZLiog

```
BurpExtender  →  
registerExtenderCallbacks()  →  
执行注册函数代码
```

registerExtenderCallbacks：执行主逻辑
getTabCaption：显示到burp上的插件名
getUiComponent：插件自定义的 UI 组件

## 入门-HelloWorld插件

参考 https://mp.weixin.qq.com/s/tEWbqAUxQXMjvNAKJD1N7g

```
<dependencies>
	<dependency>
		<groupId>net.portswigger.burp.extender</groupId>
        <artifactId>burp-extender-api</artifactId>
        <version>2.3</version>
    </dependency>
    <dependency>
        <groupId>com.github.adedayo.intellij.sdk</groupId>
        <artifactId>forms_rt</artifactId>
        <version>142.1</version>
    </dependency>
</dependencies>

```

## IDEA Ultimate版本

https://mp.weixin.qq.com/s/fQg5wXB7CmxyKqTLJpo4Rw

2、插件安装UI Designer
添加依赖，添加UI，设置编译
按钮或输入框事件监听器代码编写

3、添加UI及编写代码监听

```
Test test = new Test();
return test.$$$getRootComponent$$$();
```

## 某漏洞检测插件

```
public void actionPerformed(ActionEvent e) {
                StringBuilder responseContent = new StringBuilder();
                String urltext = UrlText.getText();
                urltext = urltext + "/minio/bootstrap/v1/verify";
                try {
                    HttpURLConnection connection = (HttpURLConnection) new URL(urltext).openConnection();
                    connection.setRequestMethod("POST");
                    connection.setConnectTimeout(5000);
                    connection.setReadTimeout(5000);
                    InputStream inputStream = connection.getInputStream();
                    if (inputStream != null) {
                        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, "UTF-8"));
                        String line;
                        while ((line = reader.readLine()) != null) {
                            responseContent.append(line);
                        }
                    }
                    if (responseContent.toString().contains("MINIO_ROOT_PASSWORD")) {
                        code.setText("存在漏洞");
                    } else {
                        code.setText("不存在漏洞");
                    }

                } catch (IOException ex) {
                    throw new RuntimeException(ex);
                }

            }
```