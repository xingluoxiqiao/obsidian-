# 什么是JWT
JWT（JSON Web Token）是一种开放标准（RFC 7519），顾名思义，它是token的一种，
用于在网络应用间安全地传输信息。它以JSON对象的形式安全地传输声明和信息，但需要注意的是，它只能保证数据的完整性，但任何人都可以获取到JWT中所携带的信息，所以通常被用作身份验证和授权的方式。
# JWT的组成
JWT由3部分组成：标头(Header)、有效载荷(Payload)和签名(Signature)。在传输的时候，会将JWT的3部分分别进行Base64编码后用  ‘.’  进行连接形成最终传输的字符串。
![[Pasted image 20240414194705.png]]
1. **头部（Header）**：包含了JWT的元数据和加密算法。通常包括两部分信息：令牌的类型（比如JWT）和所使用的加密算法（比如HMAC SHA256或RSA）。
2. **载荷（Payload）**：即需要传递的信息，例如用户的身份信息等。
3. **签名（Signature）**：对于前两部分使用Base64编码后的结果进行签名，以确保JWT的完整性和验证发送者的身份。签名的生成通常需要使用密钥，确保只有持有密钥的一方能够创建和验证JWT。
# 种类
JWT(JSON Web Token)指的是一种规范，它允许我们使用JWT在两个组织之间传递安全可靠的信息，针对不同的加密或生成算法，JWT的具体实现可以分为以下几种：  
1. nonsecure JWT：未经过签名（header部分没有指定签名算法），不安全的JWT  
2. JWS：经过特殊的签名算法生成了签名的JWT  
3. JWE：payload部分经过加密（对称加密或非对称加密）的JWT，确保其在传输过程中的保密性。JWE 由五个部分组成：
	- **头部（Header）**：包含了加密算法、加密密钥等信息，用于指示如何对 payload 进行加密。
    - **加密密钥（Encrypted Key）**：可选项，用于对 payload 进行加密的密钥，如果使用对称加密算法则会使用此项。
	- **初始化向量（Initialization Vector）**：用于对 payload 进行加密时的初始化向量，如果使用对称加密算法则会使用此项。
	- **加密的内容（Encrypted Content）**：加密后的 payload 内容。
	- **认证标签（Authentication Tag）**：用于验证加密内容的完整性和真实性，如果使用 AEAD 加密模式则会使用此项。
# 对比JWT，cookie，session
1. **JWT (JSON Web Token)**:
    - **特点**:
        - 无状态（Stateless）：JWT本身包含了用户信息和声明，服务器不需要在数据库或其他存储中保存会话状态，因此无需在多个服务器之间共享会话信息。
        - 自包含性（Self-contained）：JWT中包含了所有需要的信息，使得客户端可以直接解析和使用，减少了对服务器的查询压力。
        - 跨域（Cross-Origin）：JWT可以在不同域之间安全传输，并且可以通过设置适当的跨域策略来实现跨域通信。
    - **适用场景**:
        - 适用于分布式系统或微服务架构，无需服务器端保存会话状态。
        - 适用于需要无状态认证的场景。
2. **Cookie**:
    - **特点**:
        - 存储在客户端：Cookie是由服务器发送到客户端，并存储在客户端的浏览器中。
        - 可设置过期时间：可以通过设置Cookie的过期时间来控制用户会话的有效期。
        - 自动发送：浏览器在每次向服务器发送请求时都会自动携带相应的Cookie。
    - **适用场景**:
        - 适用于传统的Web应用程序，可以通过设置Cookie来维持用户的会话状态。
        - 可以用于存储少量敏感信息，但需要注意安全性。
3. **Session**:
    - **特点**:
        - 存储在服务器：Session数据存储在服务器端，客户端只保存了Session ID。
        - 可以存储更多信息：相比Cookie，Session可以存储更多的用户信息，但会增加服务器端的存储压力。
        - 可以根据需要进行管理：服务器可以根据需要对Session进行管理，包括设置过期时间、销毁Session等。
    - **适用场景**:
        - 适用于需要存储大量用户信息或敏感信息的场景。
        - 在需要对会话进行更多控制和管理的情况下使用。
# JWT使用示例（身份校验）
在实际的SpringBoot项目中，一般我们可以用如下流程做登录：  
1. 在登录验证通过后，给用户生成一个对应的随机token(注意这个token不是指jwt，可以用uuid等算法生成)，然后将这个token作为key的一部分，用户信息作为value存入Redis，并设置过期时间，这个过期时间就是登录失效的时间  
2. 将第1步中生成的随机token作为JWT的payload生成JWT字符串返回给前端  
3. 前端之后每次请求都在请求头中的Authorization字段中携带JWT字符串  
4. 后端定义一个拦截器，每次收到前端请求时，都先从请求头中的Authorization字段中取出JWT字符串并进行验证，验证通过后解析出payload中的随机token，然后再用这个随机token得到key，从Redis中获取用户信息，如果能获取到就说明用户已经登录
```java
public class JWTInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String JWT = request.getHeader("Authorization");
        try {
            // 1.校验JWT字符串
            DecodedJWT decodedJWT = JWTUtils.decode(JWT);
            // 2.取出JWT字符串载荷中的随机token，从Redis中获取用户信息
            ...
            return true;
        }catch (SignatureVerificationException e){
            System.out.println("无效签名");
            e.printStackTrace();
        }catch (TokenExpiredException e){
            System.out.println("token已经过期");
            e.printStackTrace();
        }catch (AlgorithmMismatchException e){
            System.out.println("算法不一致");
            e.printStackTrace();
        }catch (Exception e){
            System.out.println("token无效");
            e.printStackTrace();
        }
        return false;
    }
}
```
在实际开发中可以用下列手段来增加JWT的安全性：  
1. 使用HTTPS来传输，更加安全，因为JWT是在请求头中传递的，可以避免网络劫持
2. 保证服务器的安全，JWT的哈希签名的密钥是存放在服务端的，所以只要服务器不被攻破，理论上JWT是安全的。
3. 定期更换服务端的哈希签名密钥(相当于盐值)，避免JWT被暴力穷举破解。
# 补充
## 对称加密与非对称加密
1. **对称加密**:
    - **特点**:
        - 使用相同的密钥进行加密和解密。
        - 加密和解密速度快，效率高。
        - 密钥的管理相对简单，但需要确保密钥的安全性。
    - **应用场景**:
        - 适用于需要高效加密和解密的场景，如数据传输过程中的加密。
        - 通常用于对称加密的算法有DES、3DES、AES等。
2. **非对称加密**:
    - **特点**:
        - 使用一对密钥（公钥和私钥）进行加密和解密，公钥用于加密，私钥用于解密。
        - 加密速度相对较慢，安全性高。
        - 密钥的管理相对复杂，需要保护私钥的安全性。
    - **应用场景**:
        - 适用于安全性要求较高的场景，如数字签名、身份认证等。
        - 通常用于非对称加密的算法有RSA、DSA、ECC等。