
쿠키값을 활용해 sessionID를 주고받으며 Session의 동작방식에 대해서 알수있었다.
아래는 HTTPrequest처리 코드이다.
<details>
<summary>code</summary>
<div markdown="1">
    
```java
    
import http.HttpRequest;
import http.HttpResponse;
import http.HttpSessions;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.util.UUID;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import controller.Controller;

public class RequestHandler extends Thread {
    private static final Logger log = LoggerFactory.getLogger(RequestHandler.class);

    private Socket connection;

    public RequestHandler(Socket connectionSocket) {
        this.connection = connectionSocket;
    }

    public void run() {
        log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(),
                connection.getPort());

        try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
            HttpRequest request = new HttpRequest(in);
            HttpResponse response = new HttpResponse(out);

            if (request.getCookies().getCookie(HttpSessions.SESSION_ID_NAME) == null) {
                response.addHeader("Set-Cookie", HttpSessions.SESSION_ID_NAME + "=" + UUID.randomUUID());
            }

            Controller controller = RequestMapping.getController(request.getPath());
            if (controller == null) {
                String path = getDefaultPath(request.getPath());
                response.forward(path);
            } else {
                controller.service(request, response);
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private String getDefaultPath(String path) {
        if (path.equals("/")) {
            return "/index.html";
        }
        return path;
    }
}
```

</div>
</details>
쿠키 값으로부터 SessionID가 존재하지 않는 값이면 새로운 세션ID를 발급하고 서버에 저장한다.

쿠키의 logined = True와 같이 로그인 쿠키값을 조작하면 로그인상태가 되는것과 다르게 암호화된 SessionID를 알수없다면 데이터를 조작할수없다.

<details>
<summary>session저장소 </summary>
<div markdown="1">


```java

import java.util.HashMap;
import java.util.Map;

public class HttpSession {
    private Map<String, Object> values = new HashMap<String, Object>();

    private String id;

    public HttpSession(String id) {
        this.id = id;
    }

    public String getId() {
        return id;
    }

    public void setAttribute(String name, Object value) {
        values.put(name, value);
    }

    public Object getAttribute(String name) {
        return values.get(name);
    }

    public void removeAttribute(String name) {
        values.remove(name);
    }

    public void invalidate() {
        HttpSessions.remove(id);
    }
}
```
    
</div>
</details>
    
    
Session 저장소 객체를 만들어 객체를 통해서 세션값을 관리해줄수있다.
    
<details>
<summary>session데이터 처리 클래스</summary>
<div markdown="1">

```java

public class HttpSessions {
    public static final String SESSION_ID_NAME = "JSESSIONID";

    private static Map<String, HttpSession> sessions = new HashMap<String, HttpSession>();

    public static HttpSession getSession(String id) {
        HttpSession session = sessions.get(id);

        if (session == null) {
            session = new HttpSession(id);
            sessions.put(id, session);
            return session;
        }

        return session;
    }

    static void remove(String id) {
        sessions.remove(id);
    }
}
```
    
</div>
</details>
세션 아이디를 통해서 보안을 강화할수있지만 세션아이디의 복호화 뿐만아니라 쿠키의 다양한 속성을 활용해 보안을 강화하는것 또한 중요해보인다.
