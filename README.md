
## HttpRequest기능 분할 코드
```java
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
객체지향프로그래밍을 하기위해서 클라이언트로부터 요청을 받는 기능만을 구분해서 구현
```java
public RequestLine(String requestLine){
        log.debug("request line : {}",requestLine);
        String[] tokens = requestLine.split(" ");
        if(tokens.length!=3){
            throw new IllegalArgumentException(requestLine+"이형식이 맞지않습니다");
        }
        method = tokens[0];
        if ("POST".equals(method)){
            path = tokens[1];
            return;
        }
        int index = tokens[1].indexOf("?");
        if(index==-1){
            path = tokens[1];
        }else{
            path = tokens[1].substring(0,index);
            params = HttpRequestUtils.parseQueryString(tokens[1].substring(index+1));
        }
    }
```
요청코드를 헤더를 분리하는 기능을 httprequest에서 리팩토링
HttpResponse기능 분할 코드
```java
private static final Logger log = LoggerFactory.getLogger(HttpResponse.class);
    private DataOutputStream dos = null;
    private Map<String,String> headers = new HashMap<String,String>();

    public HttpResponse(OutputStream out){
        dos = new DataOutputStream(out);
    }

    public void addHeader(String key,String value){
        headers.put(key,value);
    }

    public void forward(String url){
        try{
            byte[] body = Files.readAllBytes(new File("./webapp"+url).toPath());
            if (url.endsWith(".css")){
                headers.put("Content-Type","text/css");
            } else if(url.endsWith(".js")){
                headers.put("Content-Type","application/javascript");
            } else{
                headers.put("Content-Type","text/html;charset=utf-8");
            }
            headers.put("Content-Length",body.length+"");
            response200Header(body.length);
            responseBody(body);
        }catch(IOException e){
            log.error(e.getMessage());
        }
    }
    public void forwardBody(String body){
        byte[] contents = body.getBytes();
        headers.put("Content-Type","text/html;charset=utf-8");
        headers.put("Content-Length",contents.length+"");
        response200Header(contents.length);
        responseBody(contents);
    }
```
클라이언트에게 응답하는 기능만을 구분해서 구현


### heroku 서버에 배포 후
* 
