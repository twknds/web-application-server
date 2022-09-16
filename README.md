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

HttpRequest기능 분할 코드
```
private static final Logger log = LoggerFactory.getLogger(HttpRequest.class);
    private String method;
    private String path;
    private Map<String,String> headers = new HashMap<String,String>();
    private Map<String,String> params = new HashMap<String,String>();
    private RequestLine requestLine;
    public HttpRequest(InputStream in){
        try{
            BufferedReader br = new BufferedReader(new InputStreamReader(in,"UTF-8"));
            String line = br.readLine();
            if(line==null){
                return;
            }
            requestLine = new RequestLine(line);
            line = br.readLine();
            while(!line.equals("")){
                log.debug("header : {}",line);
                String[] tokens = line.split(":");
                headers.put(tokens[0].trim(),tokens[1].trim());
                line=br.readLine();
            }
            if("POST".equals(method)){
                String body = IOUtils.readData(br,Integer.parseInt(headers.get("Content-Length")));
                params = HttpRequestUtils.parseQueryString(body);
            }else{
                params = requestLine.getParams();
            }
        }catch(IOException e){
            log.error(e.getMessage());
        }
    }
    ```
    *객체지향

### heroku 서버에 배포 후
* 
