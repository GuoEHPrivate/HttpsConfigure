#HTTPS配置步骤

##1.证书生成

证书生成在Linux中完成，Windows环境下可使用Git Bash工具

###1. CA证书生成
首先，模拟生成权威证书颁发机构（第三方）的证书

在命令行中依次输入以下命令：

`openssl genrsa -out ca.key 1024`

![ca1](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ca1.PNG)

`openssl req -new -key ca.key -out ca.csr`

- 此处和下面生成服务端、客户端证书时要求填写国家代码、公司名称等，尤其`Organization Name`和`Organization Unit`不应相同。

- `Common Name`应该填写域名（e.g. `baidu.com`）或者IP。

![ca2](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ca2.PNG)

`openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt`

- `days`单位为*天*

![ca3](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ca3.PNG)

此时，CA机构证书生成完成。

###2. Server证书生成

Server证书生成时需要使用CA证书进行签名。

`openssl genrsa -out server.key 1024`

![server1](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/server1.PNG)

`openssl rsa -in server.key -pubout -out server.pem`

![server2](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/server2.PNG)

`openssl req -new -key server.key -out server.csr`

- 此处一样要填写国家代码、公司名称等，注意不要与CA签发时的`Organization Name`和`Organization Unit`相同。

![server3](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/server3.PNG)

`openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt`

![server4](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/server4.PNG)

此时，server证书生成完成。

###3. Client证书生成

`openssl genrsa -out client.key 1024`

![client1](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/client1.PNG)

`openssl rsa -in client.key -pubout -out client.pem`

![client2](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/client2.PNG)

`openssl req -new -key client.key -out client.csr`

- 与生成server时相同，此处一样要填写国家代码、公司名称等，注意不要与CA签发时的`Organization Name`和`Organization Unit`相同。

![client3](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/client3.PNG)

此时，client证书生成完成。

##2.Nginx配置

为Nginx的配置文件中添加以下代码，监听443端口：

    server {
    	listen 443;
    	ssl on;
    	ssl_certificate /usr/local/nginx/conf/server.crt;
    	ssl_certificate_key /usr/local/nginx/conf/server.key;
    }

![nginx1](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/nginx1.PNG)

然后在服务器命令行中输入以下命令以重启Nginx服务器：

`nginx -s reload`

![nginx2](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/nginx2.PNG)

##3.Tomcat配置

TODO

##4.浏览器配置

待服务器重启后，即可在地址栏中输入以`https`开头的地址查看。此时，不需要再输入端口号，因为以`https`开头的接口都会默认访问`443`端口。由于是自签名的证书，不在浏览器信任的证书中，所以浏览器会提示安全证书存在问题，但是点击`继续浏览此网站`也可以继续查看：

![ie1](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ie1.png)

但是再地址栏中会出现红色的提示：

![iepre2](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/iepre2.png)

开始将第一步签发的CA的证书安装到本地：

在IE浏览器中，依次选择`工具`-`Internet选项`-`内容`-`证书`-`受信任的根证书颁发机构`-`导入`，选择`ca.crt`证书文件，按照提示直到证书安装完成

![ie2-2](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ie2-2.PNG)

![ie3](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ie3.PNG)

![ie4](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ie4.PNG)

![ie5](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ie5.PNG)

![ie6](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ie6.PNG)

安装完成后，重启IE浏览器，再次打开刚才的链接，此时不会再出现安全提示。

![ie7](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/ie7.PNG)

PC端证书安装完成。

##5.Android配置

在网上查到的资料中，都提到了Android中只支持BKS格式的证书而不支持CRT格式的证书，因此需要将CRT格式的证书转换成BKS格式。转换时需要`bcprov-ext-jdk15on-150.jar`的Jar包。

将Jar包放到CRT证书所在的文件夹下（仅仅为了输入方便），在命令行中输入以下命令：

    keytool -importcert -v -trustcacerts -alias client -file server.crt -keystore server.bks -storetype BKS 
     -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath bcprov-ext-jdk15on-150.jar 
     -storepass cafsc@client`

- `-alias`后为别名，暂不知道其用途

- `-file`后为需要转换的CRT证书文件名

- `-keystore`为转换后BKS证书的文件名

- `-storetype`为需要转换为BKS格式

- `-providerpath`为Jar包的文件名

- `-storepass`自行设置一个密码

当询问`是否信任此证书?`时，输入`y`，完成格式转换。

![transport1](https://github.com/GuoEHPrivate/HttpsConfigure/raw/init/res/transport1.PNG)

但是，在实际使用时，发现BKS格式的证书并不能使用，反而CRT格式的证书可以直接使用，具体原因未知。因此，下面的方法将介绍CRT格式证书的使用。

在使用时直接将`server.crt`导入到工程的`raw`文件夹下。

在使用`OkHttp`网络库构建`OkHttpClient.Builder`对象时，加入以下代码：

`builder.sslSocketFactory(getSSLSocketFactory(MyApplication.getContext()));`

具体的`getSSLSocketFactory()`代码块如下：

    import java.io.IOException;
    import java.io.InputStream;
    import java.security.KeyManagementException;
    import java.security.KeyStore;
    import java.security.KeyStoreException;
    import java.security.NoSuchAlgorithmException;
    import java.security.SecureRandom;
    import java.security.cert.Certificate;
    import java.security.cert.CertificateException;
    import java.security.cert.CertificateFactory;
    
    public final class NetManager {

    	····

    	private static SSLSocketFactory getSSLSocketFactory(Context context) {
        	if (context == null) {
            	throw new NullPointerException("context == null");
        	}
        	//CertificateFactory用来证书生成
        	CertificateFactory certificateFactory;
        	try {
            	certificateFactory = CertificateFactory.getInstance("X.509");
            	//读取本地证书
            	InputStream is = context.getResources().openRawResource(R.raw.server);
            	Certificate cert = certificateFactory.generateCertificate(is);
            	if (is != null) {
                	is.close();
            	}
            	//Create a KeyStore containing our trusted CAs
            	KeyStore keyStore = KeyStore.getInstance("BKS");
            	keyStore.load(null, null);
            	keyStore.setCertificateEntry("cert", cert);
            	//Create a TrustManager that trusts the CAs in our keyStore
            	TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            	trustManagerFactory.init(keyStore);
            	//Create an SSLContext that uses our TrustManager
            	SSLContext sslContext = SSLContext.getInstance("TLS");
            	sslContext.init(null, trustManagerFactory.getTrustManagers(), new SecureRandom());
            	return sslContext.getSocketFactory();
        	} catch (KeyStoreException e) {
            	e.printStackTrace();
        	} catch (CertificateException e) {
           		e.printStackTrace();
        	} catch (NoSuchAlgorithmException e) {
            	e.printStackTrace();
        	} catch (IOException e) {
            	e.printStackTrace();
        	} catch (KeyManagementException e) {
            	e.printStackTrace();
        	}
        	return null;
    	}

    ····

    }

Android端证书添加完成。