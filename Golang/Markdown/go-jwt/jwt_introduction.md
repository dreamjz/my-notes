# JWT

## 1.1 跨域认证问题

传统的session的认证流程如下:

	1. 用户向服务器发送用户名和密码
	2. 服务器验证通过后,将相关信息保存至session中
	3. 服务器将session ID返回给客户端并存储至cookie中
	4. 用户请求时会附上session ID,服务器会根据session ID来得知用户身份

### 1.1.1 传统session认证的问题

**Session**:        每个用户经过服务认证之后,会将session信息保存至内存中,当用户的数量增加时服务器的负荷会增大

**扩展性(Scaling)**: 在分布式系统中,用户需在保存其session信息的服务器中才可以获取授权,限制了应用的扩展能力

**CSRF:**           因为是基于cokkie进行识别的,若cokkie被截获,用户容易受到跨站请求攻击

### 1.1.2 基于Token的鉴权机制

基于token的认证和传统的session方式不同,token信息存放在客户端而不是服务端

其认证流程如下:

 	1. 用户向服务器发送用户名和密码
 	2. 服务器验证用户信息
 	3. 验证通过后向用户签发token
 	4. 用户请求时附上token,服务器验证token来进行授权

token在每次请求时发送给服务端,其应放在请求头中并要求服务端支持CORS(跨域资源共享)

## 1.2 JWT

> Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（[(RFC 7519](https://link.jianshu.com?t=https://tools.ietf.org/html/rfc7519)).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

### 1.2.1 JWT的构成

JWT由三个部分构成并有点号(.)连接:

1. Header    头部
2. Payload   负载
3. Signature签名

完整结构如下:

```
Header.Payload.Signature
```

#### 1.2.1.1 Header

Header部分是一个JSON对象:

```JSON
{
    "alg":"HS256", 
    "typ":"JWT"
}
```

其中:

- alg(algorithm) : 签名的算法,默认为HMAC SHA256(HS256)
- typ(tpye)      : token的类型,JWT令牌为JWT

之后将Header使用base64URL算法转为字符串

#### 1.2.1.2 Payload

