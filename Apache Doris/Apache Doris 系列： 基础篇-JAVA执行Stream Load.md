简介
----

Stream Load 的本质是调用 Doris BE 节点的 HTTP API 来实现数据的导入，因为我们可以用JAVA HTTP CLIENT 来实现数据的导入。


代码
----
```java
public class StreamLoadExample {
    private final static String HOST = "192.168.56.104"; // FE IP
    private final static int PORT = 8030; //FE HTTP 端口
    private final static String DATABASE = "test"; // 数据库名
    private final static String TABLE = "order_info_example"; // 数据表名
    private final static String USER = "test"; // Doris 用户名
    private final static String PASSWD = "password123"; // Doris 密码
    private final static String LOAD_FILE_NAME = "src/main/resources/data.txt"; // 本地文件路径

    private final static String loadUrl = String.format("http://%s:%s/api/%s/%s/_stream_load",
            HOST, PORT, DATABASE, TABLE);

    private final static HttpClientBuilder httpClientBuilder = HttpClients
            .custom()
            .setRedirectStrategy(new DefaultRedirectStrategy() {
                @Override
                protected boolean isRedirectable(String method) {
                    // If the connection target is FE, you need to handle 307 redirect.
                    return true;
                }
            });

    public void load(File file) throws Exception {
        try (CloseableHttpClient client = httpClientBuilder.build()) {
            HttpPut put = new HttpPut(loadUrl);
            put.setHeader(HttpHeaders.EXPECT, "100-continue");
            put.setHeader(HttpHeaders.AUTHORIZATION, basicAuthHeader(USER, PASSWD));

            // 设置stream load 的label, 用于避免重复导入数据
            put.setHeader("label","label-example");
            //设置字段分隔符
            put.setHeader("column_separator",",");

            // Set the import file.
            // StringEntity can also be used here to transfer arbitrary data.
            FileEntity entity = new FileEntity(file);
            put.setEntity(entity);

            try (CloseableHttpResponse response = client.execute(put)) {
                String loadResult = "";
                if (response.getEntity() != null) {
                    loadResult = EntityUtils.toString(response.getEntity());
                }

                final int statusCode = response.getStatusLine().getStatusCode();
                if (statusCode != 200) {
                    throw new IOException(
                            String.format("Stream load failed. status: %s load result: %s", statusCode, loadResult));
                }

                System.out.println("Get load result: " + loadResult);
            }
        }
    }

    private String basicAuthHeader(String username, String password) {
        final String tobeEncode = username + ":" + password;
        byte[] encoded = Base64.encodeBase64(tobeEncode.getBytes(StandardCharsets.UTF_8));
        return "Basic " + new String(encoded);
    }

    public static void main(String[] args) throws Exception{
        StreamLoadExample loader = new StreamLoadExample();
        File file = new File(LOAD_FILE_NAME);
        loader.load(file);
    }
}

```

[github代码地址](https://github.com/baiyuelanshan/apache-doris-example/blob/main/src/main/java/StreamLoadExample.java)
