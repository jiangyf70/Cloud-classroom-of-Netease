# 网易云课堂视频下载  
网易云课堂视频下载（包含原理介绍，在最后），收费视频需要**购买过**的才能下载
原大神的项目已经不能使用，重新修改可用了。



## 测试环境  
win10 + python3.7.5



## 使用方法  



> **注意**，这只是为了下载网易云课堂收费视频，解密并下载，并不是一个完善和长期的工具，不考虑使用的方便，请严格按照使用说明操作或依据原理自行完善修改



1. 安装依赖的第三方库  

    ```cmd
    pip install -r requirements.txt
    ```
    **Crypto** 所在的第三方库文件夹记得Crypto首字母要大写，否则Crypto提示出错。

    > `pycryptodome`用以进行AES解密  
    > `mitmproxy`用以拦截请求  
    > 其他一般的像`requests`等第三方库如有缺失请自行安装  

    请参照网上的教程**安装mitmproxy证书**，不然抓不到https包！！！  
    请参照网上的教程**安装mitmproxy证书**，不然抓不到https包！！！  
    请参照网上的教程**安装mitmproxy证书**，不然抓不到https包！！！  
https://www.jianshu.com/p/036e5057f0b9?ivk_sa=1024320u
三、mitmproxy证书配置
mitmproxy和Charles软件一样，需要进行证书配置，为什么要进行证书配置？由于我们要抓取的大部分App页面都采用了HTTPS，也就是经过加密，若没有证书，就无法识别抓取的数据。另外，需要注意的是，mitmproxy与Charles软件一样需要在PC和手机端安装证书。

1、首先cmd运行一下以下命令，以便在windows系统中产生CA证书

mitmdump
2、接着在.mitmproxy目录找到CA证书，如下图所示

image
3、双击mitmproxy-ca.p12进入导入证书的页面，点击“下一步”

image
4、不用输入密码，直接“下一步”

image
5、选择“将所有的证书都放入下列存储”，接着选择“受信任的根证书颁发机构”

image
6、最后，弹出警告窗口，直接点击“是”

image

2. 启动脚本  

    ```cmd
    python main.py
    ```

    如果没有报错，那么恭喜你，你的脚本**应该**是可以正常使用的。  



3. 设置代理  
    - 如果你没有对脚本做过修改，那么**默认**的代理地址为本机的8080端口，即`127.0.0.1:8080`。  
    - 打开你电脑的`网络设置` -> `代理`，开启代理服务，地址为`127.0.0.1`，端口为`8080`，注意**点击保存才能生效**。如果你找不到相关设置或不会设置，请自行询问度娘*电脑如何设置网络代理*。以下为win10代理设置截图：  
    ![win10代理](./images/proxy.png)  



4. 获取视频信息  
    - 保证你的代理设置正确且生效，同时保持脚本的运行。  
    - 打开网易云课堂中你要下载的视频，让其播放，此时，你就能看见脚本运行界面显示**已记录**了该视频的相关信息。  
    ![抓取信息](./images/running.png)  
    > **注意**，这里的命名为**视频的名称**（即用户上传视频时视频文件的名称），而**非课程的名称**，所以，如果视频上传者瞎起名字的话，你看到的也可能是一堆乱七八槽的东西哈哈  
    - 如果脚本没有报错，也没有获取到任何东西，请检查你的代理。这是很好检查的，如果你的**代理设置未生效**，在脚本未运行的情况下，你是不能上网的。只有**代理和脚本全都正常生效**，你才能正常上网。而如果脚本正常运行了你还是上不了网，那就说明你的**代理设置错误**。  



5. 下载视频  
    - **关闭**你的代理和脚本，使电脑恢复到正常状态。
    - 运行下载脚本
    ```cmd
    python down163.py
    ```
    - 下载脚本会根据之前记录的相关信息，下载你的视频。
    > **注意**，为了避免重复下载，你应该及时清理`msgs`和`keys`中的文件。在每次**获取信息前**，先将它们清空或换个地方保存吧。



## 原理介绍  
### 视频分析  
1. 网易云课堂的视频一共分为三大类：  
 
    - 收费视频的ts文件都是经过了加密的。前端在播放视频前，会先请求一个key文件，这里面存放的就是二进制的解密key，然后再用key对每个ts进行解密播放。

2. 痛点：  
    - 三类视频中前两种只靠抓包分析请求就能很轻易的解决。关键是第三种加密视频，虽然我们靠分析理清了全过程，但key文件仍然不是那么好拿的。
    - key文件的地址存在m3u8文件中，但它是个动态的地址，每次拿到的都不一样。且这个地址只能请求一次还有时限限制。
    - 分析js是个体力活，反爬的本质就是提高爬虫的成本（只要在前端，花时间总是能破解的）。为了一时所需去阅读大量的js代码，性价比太低了，所以我选择让浏览器代劳，我们直接拿结果就行。



### 拦截请求  
mitmproxy是一个代理模块，会在你指定的端口开启一个代理服务，所用通过这个端口的请求和响应都会被记录，你可以通过设置代理脚本对它们进行随意的改造，具体可以查看官方文档。  
所以，这个问题的解决方案就是通过mitmproxy获取我们所需的信息（比如key），存储后再进行下载。  
在`intercapters.py`文件的最后，你可以看到我一共加载了2个拦截器：  
    - `NameIntercapter`: 获取视频名称，及m3u8的url  
    - `KeyIntercapter`: 获取解密所需的key  
有了这2个拦截器，我们就能获取到视频下载所需要的信息。  
![抓取位置](./images/a.png)  
![抓取key](./images/b.png)  

