# 后台开发初体验

标签（空格分隔）： Java

> 原文 https://www.zybuluo.com/Tyhj/note/585487

自己写Android也已经两年了，平时和其他同学合作都是别人写后台，或者直接用第三方的Api，把数据存到别人服务器上面，自己也了解过后台就是自己从来没有去实现过，所以现在感觉没什么事情，就想要自己搞一下。
之前考虑过用**Python**，但是还要去学习好久的基本语法，自己也是懒得去学了，还是来用**Java**好了。

## 基本数据传输

### 环境配置
环境配置什么的我就不想说了，很简单
把Android端作为客户端，用eclipse或者Myeclipse写服务器端，需要环境：

- [x] Android studio
- [x] eclipse
- [ ] Myeclipse
- [x] Tomcat

反正能写Java,Android和Jsp,再安装个Tomcat就好了,Myeclipse应该自带了Tomcat，但是需要破解，其他三个是开源的。

### 实现服务器接收和反馈信息

*新建一个web project工程*

#### 新建一个servlet
*在写java的文件夹下面选择新建一个servlet，新建好了以后应该是已经继承了**HttpServlet**并且重写了doGet和doPost方法*

#### 配置servlet
*新建了servlet后现在还不能通过url来直接访问，必须先在web.xml里面配置后才可以访问，我的配置如下*
```xml
<servlet>
    <servlet-name>sign_up</servlet-name>
    <servlet-class>servlet.Signup</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>sign_up</servlet-name>
    <url-pattern>/talk/sign_up</url-pattern>
  </servlet-mapping>
```
> 在WebContent/WEB-INF下面，如果找不到就是创建工程的时候没有选择生成，需要重新创建工程并选择生成web.xml，每个软件或者版本都可能不一样

就是两个标签，
其中servlet-name必须一样，servlet-class就是这个类的路径，就是包名.类名，
url-pattern就是url有关，可以随便写，比如我的要访问这个servlet，url就是
>http://192.168.31.215:8080/Talk/talk/sign_up。

其中192.168.31.215是你的ip地址
8080是端口,tomcat端口一般是8080
Talk是项目名

本地测试的，让电脑和手机连接同一个WiFi，然后在终端输入，就看到了**ip地址**，
```
mac：ifconfig
在en0下面：
inet 192.168.31.215 netmask 0xffffff00 broadcast 
```
```
windows：ipconfig
应该是IPv4:
无线局域网适配器 WLAN:
   IPv4 地址 . . . . . . . . . . . . : 192.168.31.104
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 192.168.31.1

```

#### 接收数据
在doGet和doPost里面接收数据，具体在那个里面自己随便写，会决定客户端请求的方式，可以先看一下Http协议的get和post的区别。

接收格式为：
```java
request.setCharacterEncoding("UTF-8");
		response.setCharacterEncoding("UTF-8");
		String username = request.getParameter("username");
		String password = request.getParameter("password");
```
其中username和password就是在客户端发来的数据。
>Android studio的编码是UTF-8，所以我们设置编码格式避免乱码

#### 返回数据给客户端
返回数据给客户端，好让客户端知道请求成功没有，或者返回所请求的资源。这里一般是要用一种固定格式的，以便于客户端好解析，所以我们这里用json格式来包装数据，当然需要导入jar包，下载，复制到项目lib目录，然后，build to path就好了，我用的是google-gson，下载地址为：
> http://repo1.maven.org/maven2/com/google/code/gson/gson/2.3.1/gson-2.3.1.jar

现在随便返回一些值，把数据封装到json中：
```java
JsonObject json = new JsonObject();
		json.addProperty("code", "1");
		json.addProperty("name", username);
		json.addProperty("pas", password);

```
然后把json转成字符串
```java
String postJson =json.toString();
```

最后返回给客户端
```java
		PrintWriter out = response.getWriter();
		out.write(postJson);
		out.close();
```
> 这样的过程应该是比较合理的，客户端拿到数据后就可以轻松地解析出数据，开头一般设置一个状态码，让用户知道请求是不是成功了。



### Android端发送请求并获得返回值
新建一个Android项目

#### 发起请求
新建一个类叫Myhttp,用于保存一些发送http请求的方法
>在Android中有几种不同的方法发起http请求，我会用**HttpURLConnection**方法来请求。post和get请求的方法也是不一样的

##### Post请求方法，获取返回的数据并转成Json格式
```java
    public static JSONObject getJson_Post(String data, String url) {
        HttpURLConnection conn = null;
        URL mURL = null;
        try {
            mURL = new URL(url);
            conn = (HttpURLConnection) mURL.openConnection();
            conn.setRequestMethod("POST");
            conn.setReadTimeout(5000);
            conn.setConnectTimeout(10000);
            conn.setDoOutput(true);
            OutputStream out = conn.getOutputStream();
            out.write(data.getBytes());
            out.flush();
            out.close();
            int responseCode = conn.getResponseCode();// 调用此方法就不必再使用conn.connect()方
            if (responseCode == 200) {
            //获取到的返回值
                InputStream is = conn.getInputStream();
            //将返回值转化成字符串
                String state = getStringFromInputStream(is);
                Log.e("Tag",state);
            //将字符串转化成Json格式
                JSONObject jsonObject = new JSONObject(state);
                if (jsonObject != null)
                    return jsonObject;
            }
            return null;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
```

##### Post请求方法，获取返回的数据并转成Json格式
```java
    public static JSONObject getJson_Get(String data, String url) {
        HttpURLConnection conn = null;
        URL mURL = null;
        try {
            mURL = new URL(url + "?" + data);
            conn = (HttpURLConnection) mURL.openConnection();
            conn.setRequestMethod("GET");
            conn.setReadTimeout(5000);
            conn.setConnectTimeout(10000);
            InputStream is = conn.getInputStream();
            String state = getStringFromInputStream(is);
            //Log.e("Tag",state);
            JSONObject jsonObject = new JSONObject(state);
            if (jsonObject != null)
                return jsonObject;
            return null;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
```

这是基本的http请求，我们可以在任何Post和Get请求调用它，我们请求成功后拿到返回值再根据实际功能进行解析，其中获取到的字符串是这个样子的：
```json
{"code":"1","name":"嗨","pas":"你好"}
```

#### 调用方法和解析数据
首先要开启访问网络权限，然后开启线程，在线程中执行，get就是执行doGet,Post同理。
其中data就是你要传给服务器的参数，url就是地址。
```java
String url="http://192.168.31.215:8080/Talk/talk/sign_up";
new Thread(new Runnable() {
            @Override
            public void run() {
                JSONObject jsonObject=MyHttp.getJson_Post("username=嗨&password=你好",url);
                try {
                    String code=jsonObject.getString("code");
                    String name=jsonObject.getString("name");
                    String pas=jsonObject.getString("pas");
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        }).start();
```
这样就完成了基本的数据传输。


## 文件上传

### 服务器端
使用Apache开发的文件上传处理库Commons FileUpload来进行文件的上传，这个库还可以，有很多优点，上传的文件可大可小。
需要这俩jar包
commons-fileupload.jar：
> 下载地址：http://apache.fayea.com//commons/fileupload/binaries/commons-fileupload-1.3.2-bin.zip

commons-io.jar
> 下载地址：http://mirrors.hust.edu.cn/apache//commons/io/binaries/commons-io-2.5-bin.zip

#### 新建servlet
新建一个servlet，然后在web.xml中配置好
```xml
<servlet>
    <servlet-name>upload</servlet-name>
    <servlet-class>servlet.Upload</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>upload</servlet-name>
    <url-pattern>/talk/setHeadImage</url-pattern>
  </servlet-mapping>
```
#### 在doPost中完成文件的读取和保存
```java
System.out.println(name + "doPost+被访问");	
		
		request.setCharacterEncoding("UTF-8");
		response.setCharacterEncoding("UTF-8");
		
		//获得磁盘文件条目工厂。  
        DiskFileItemFactory factory = new DiskFileItemFactory();  
        //获取文件上传需要保存的路径，upload文件夹需存在。  
        String path ="/Users/Tyhj/softWare/upload";
        //设置暂时存放文件的存储室，这个存储室可以和最终存储文件的文件夹不同。因为当文件很大的话会占用过多内存所以设置存储室。  
        factory.setRepository(new File(path));  
        //设置缓存的大小，当上传文件的容量超过缓存时，就放到暂时存储室。  
        factory.setSizeThreshold(1024*1024);  
        //上传处理工具类（高水平API上传处理？）  
        ServletFileUpload upload = new ServletFileUpload(factory);  
          
        try{  
            //调用 parseRequest（request）方法  获得上传文件 FileItem 的集合list 可实现多文件上传。  
            List<FileItem> list = (List<FileItem>)upload.parseRequest(request);  
            for(FileItem item:list){  
                //获取表单属性名字。  
                String name = item.getFieldName();  
                //如果获取的表单信息是普通的文本信息。即通过页面表单形式传递来的字符串。  
                if(item.isFormField()){  
                    //获取用户具体输入的字符串，  
                    String value = item.getString();  
                    request.setAttribute(name, value);  
                }  
                //如果传入的是非简单字符串，而是图片，音频，视频等二进制文件。  
                else{   
                    //获取路径名  
                    String value = item.getName();  
                    //取到最后一个反斜杠。  
                    int start = value.lastIndexOf("\\");  
                    //截取上传文件的 字符串名字。+1是去掉反斜杠。  
                    String filename = value.substring(start+1);  
                    request.setAttribute(name, filename);  
                      
                    /*第三方提供的方法直接写到文件中。 
                     * item.write(new File(path,filename));*/  
                    //收到写到接收的文件中。  
                    OutputStream out = new FileOutputStream(new File(path,filename));  
                    InputStream in = item.getInputStream();  
                      
                    int length = 0;  
                    byte[] buf = new byte[1024];  
                    System.out.println("获取文件总量的容量:"+ item.getSize());  
                      
                    while((length = in.read(buf))!=-1){  
                        out.write(buf,0,length);  
                    }  
                    in.close();  
                    out.close();  
                    
                }  
            }  
        }catch(Exception e){  
            e.printStackTrace();  
        } 
        
        JsonObject json = new JsonObject();
		json.addProperty("code", "1");
		json.addProperty("name", "上传成功");
		
		String postJson =json.toString();
		System.out.println(postJson);

		PrintWriter out = response.getWriter();
		out.write(postJson);
		out.close();
```

### Android端
Android端也是比较简单，新建一个upLoad方法来执行文件上传的操作
```java
//上传文件
    public static String upLoad(String url, File file) {
        String BOUNDARY = UUID.randomUUID().toString(); // 边界标识 随机生成
        String PREFIX = "--", LINE_END = "\r\n";
        String CONTENT_TYPE = "multipart/form-data"; // 内容类型
        HttpURLConnection conn = null;
        java.net.URL mURL = null;
            try {
                mURL = new URL(url);
                conn = (HttpURLConnection) mURL.openConnection();
                conn.setRequestMethod("POST");
                conn.setReadTimeout(5000);
                conn.setConnectTimeout(10000);
                conn.setDoInput(true); // 允许输入流
                conn.setDoOutput(true); // 允许输出流
                conn.setUseCaches(false); // 不允许使用缓存
                conn.setRequestProperty("connection", "keep-alive");
                conn.setRequestProperty("Content-Type", CONTENT_TYPE + ";boundary="+ BOUNDARY);
                OutputStream out = conn.getOutputStream();


                DataOutputStream dos = new DataOutputStream(out);
                StringBuffer sb = new StringBuffer();
                sb.append(PREFIX);
                sb.append(BOUNDARY);
                sb.append(LINE_END);


                /**
                 * 这里重点注意： name里面的值为服务器端需要key 只有这个key 才可以得到对应的文件
                 * filename是文件的名字，包含后缀名
                 */

                sb.append("Content-Disposition: form-data; name=\"file\"; filename=\""
                        + file.getName() + "\"" + LINE_END);

                sb.append("Content-Type: application/octet-stream; charset="
                        + CHARSET + LINE_END);

                sb.append(LINE_END);
                dos.write(sb.toString().getBytes());
                InputStream is = new FileInputStream(file);
                byte[] bytes = new byte[1024];
                int len = 0;
                while ((len = is.read(bytes)) != -1) {
                    dos.write(bytes, 0, len);
                }
                is.close();
                dos.write(LINE_END.getBytes());
                byte[] end_data = (PREFIX + BOUNDARY + PREFIX + LINE_END).getBytes();
                dos.write(end_data);
                dos.flush();


                int responseCode = conn.getResponseCode();// 调用此方法就不必再使用conn.connect()方
                if (responseCode == 200) {
                    InputStream input = conn.getInputStream();
                    String state = For2mat.getInstance().inputStream2String(input);
                    //Log.e("Tag",state);
                    return state;
                }
                return null;
            } catch (Exception e) {
                e.printStackTrace();
                return null;
            }
    }
```
之前使用过把文件转化成字符串来上传的方法，但是上传的文件大小有限制，还容易出错，所以这个方法还可以，我也是网上找的代码，没有深入学习过，我测试上传了一个MV,是可以用的，当然我的代码有很多需要优化的地方，服务器返回值应该在文件保存之前返回，不然文件太大的话，会造成请求超时。
> 参考文章：http://blog.csdn.net/crazy__chen/article/details/41958701