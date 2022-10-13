# 실습을 위한 개발 환경 세팅
* https://github.com/slipp/web-application-server 프로젝트를 자신의 계정으로 Fork한다. Github 우측 상단의 Fork 버튼을 클릭하면 자신의 계정으로 Fork된다.
* Fork한 프로젝트를 eclipse 또는 터미널에서 clone 한다.
* Fork한 프로젝트를 eclipse로 import한 후에 Maven 빌드 도구를 활용해 eclipse 프로젝트로 변환한다.(mvn eclipse:clean eclipse:eclipse)
* 빌드가 성공하면 반드시 refresh(fn + f5)를 실행해야 한다.

# 웹 서버 시작 및 테스트
* webserver.WebServer 는 사용자의 요청을 받아 RequestHandler에 작업을 위임하는 클래스이다.
* 사용자 요청에 대한 모든 처리는 RequestHandler 클래스의 run() 메서드가 담당한다.
* WebServer를 실행한 후 브라우저에서 http://localhost:8080으로 접속해 "Hello World" 메시지가 출력되는지 확인한다.

# 각 요구사항별 학습 내용 정리
* 구현 단계에서는 각 요구사항을 구현하는데 집중한다. 
* 구현을 완료한 후 구현 과정에서 새롭게 알게된 내용, 궁금한 내용을 기록한다.
* 각 요구사항을 구현하는 것이 중요한 것이 아니라 구현 과정을 통해 학습한 내용을 인식하는 것이 배움에 중요하다. 

## servlet interface구현 
```
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public interface Controller {
    String execute(HttpServletRequest req, HttpServletResponse resp) throws Exception;
}
```
이를 통해서 Controllerinterface를 상속받는 기능별 controller를 구현할수 있었다.

```
import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private static final String DEFAULT_REDIRECT_PREFIX = "redirect:";

    private RequestMapping rm;

    @Override
    public void init() throws ServletException {
        rm = new RequestMapping();
        rm.initMapping();
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String requestUri = req.getRequestURI();
        logger.debug("Method : {}, Request URI : {}", req.getMethod(), requestUri);

        Controller controller = rm.findController(requestUri);
        try {
            String viewName = controller.execute(req, resp);
            move(viewName, req, resp);
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }

    private void move(String viewName, HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        if (viewName.startsWith(DEFAULT_REDIRECT_PREFIX)) {
            resp.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));
            return;
        }

        RequestDispatcher rd = req.getRequestDispatcher(viewName);
        rd.forward(req, resp);
    }
}

```
HttpServlet을 상속받아 코딩한 디스패처서블렛이다.

@WebServlet 어노테이션을 통해서 /로 매핑된 .jsp를 제외한 모든 URL이 디스패처서블릿으로 연결된다. 

해당 디스패처 서블릿은 정적자원만을 처리하도록 코딩되어있으며

아래 필터 코드를 통해서 확인할수 있다.

```
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@WebFilter("/*")
public class ResourceFilter implements Filter {
    private static final Logger logger = LoggerFactory.getLogger(ResourceFilter.class);
    private static final List<String> resourcePrefixs = new ArrayList<>();
    static {
        resourcePrefixs.add("/css");
        resourcePrefixs.add("/js");
        resourcePrefixs.add("/fonts");
        resourcePrefixs.add("/images");
        resourcePrefixs.add("/favicon.ico");
    }

    private RequestDispatcher defaultRequestDispatcher;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        this.defaultRequestDispatcher = filterConfig.getServletContext().getNamedDispatcher("default");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        String path = req.getRequestURI().substring(req.getContextPath().length());
        if (isResourceUrl(path)) {
            logger.debug("path : {}", path);
            defaultRequestDispatcher.forward(request, response);
        } else {
            chain.doFilter(request, response);
        }
    }

    private boolean isResourceUrl(String url) {
        for (String prefix : resourcePrefixs) {
            if (url.startsWith(prefix)) {
                return true;
            }
        }
        return false;
    }

    @Override
    public void destroy() {
    }

}

```
css나 js등 정적 자원의 처리의 경우는 디스패처 서블릿으로 가기전 톰캣 Default Servlet으로 통하도록 필터를 통해서 가로채고 있다.

위의 디스패처서블릿 코드를 통해서 Default Servlet을 재정의 하고있고 JSP파일의 요청을 제외한 모든 요청을 디스패처서블릿으로 가도록 처리해놨다.

jsp파일은 디스패처서블릿을 통해서 매핑을 하지 않는이유는 jsp파일로 연결된 uri의 경우에는 특별한 비즈니스로직을 요구하지 않는대 컨트롤러를 생성하는 것은 객체지향적이지 않다고 생각해 

바로 JSP뷰로 연결하는 방식으로 설계하였다.

이와같이 디스패처 서블릿을 통해서 구현한 MVC패턴을 프론트 컨트롤러 패턴이라고 하며, 각컨트롤러 앞에서 프론트 컨트롤러가 모든 작업을 위임하는 방식이다.
