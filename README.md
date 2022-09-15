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

### 요구사항 1 - http://localhost:8080/index.html로 접속시 응답
* 
```java
InputStreamReader reader = new InputStreamReader(in);
            BufferedReader bufferedReader = new BufferedReader(reader);//
            String line = bufferedReader.readLine();
            if (line == null){return;}
            /*while(!"".equals(line)){
                line = bufferedReader.readLine();
                System.out.println(line);
            }*/
            String[] tokens = line.split(" ");
            byte[] body;
            if (tokens[1].equals("/index.html")==true){
                String url = tokens[1];
                body = Files.readAllBytes(new File("./webapp"+url).toPath());
            }
```
요구사항을 해결하면서 작성한 코드이다.
기존에 소켓통신에 대한 이해가 부족하여서 처음 요구사항을 받았을때 매우 막막하였다.
통신할때 HTTP헤더가 인풋스트림을 통해서 들어온다는 사실또한 새롭게 알게되었다.
인풋스트림을 가공하여 Mapping의 가동원리를 알게되었다.

### 요구사항 2 - get 방식으로 회원가입
* 
```java
                int index = url.indexOf("?");
                String queryString = url.substring(index+1);
                Map<String, String> param = HttpRequestUtils.parseQueryString(queryString);
                User user = new User(param.get("userId"),param.get("password"),param.get("name"),param.get("email"));
```
GET 으로 회원가입 요청이 오게되면 http 헤더에 정보가 붙어서 오게되는대 이 헤더값을 파싱해서 회원가입 정보를 얻어내는 방식을 사용하였다.
GET요청이 들어오게 되면 기존에 /index.html 리다이렉트와 마찬가지로 url(회원가입)뒤에 ?가 붙고 그뒤에 정보가 붙어서 나오는 방식이었다.
그뒤에는 단순히 split을 통해서 뒷정보를 추출하고 parseQueryString을 이용하여 자동으로 정보가 구분되었다.

### 요구사항 3 - post 방식으로 회원가입
* 
```java
                String queryString = util.IOUtils.readData(bufferedReader, cl);
                Map<String, String> param = HttpRequestUtils.parseQueryString(queryString);
                User user = new User(param.get("userId"),param.get("password"),param.get("name"),param.get("email"));
```
이거는 위의 GET방식에서 단순히 POST형식으로 바뀌면서 Header에 붙어오던 정보들이 Body에 붙어오게 되었다.
Body부를 읽기위해서는 Content-Length가 필요해서 Line을 읽으면서 Context-Length부를 찾아내서 그값을 추출해야했다.
추출한뒤 이를 readData에 넣고 돌리면 queryString이 나오는 방식, 후에는 위의 GET방식과 동일한 절차로 진행되었다.
### 요구사항 4 - redirect 방식으로 이동
* 
```java
                DataOutputStream dos = new DataOutputStream(out);
                response302Header(dos, "/index.html");
                -------------------------------------------------
    private void response302Header(DataOutputStream dos, String url) {
        try {
            dos.writeBytes("HTTP/1.1 302 Redirect \r\n");
            dos.writeBytes("Location: "+url+" \r\n");
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }                
```
url값만 바꿔줄시에는 body부에 저장된정보가 사라지지않기 때문에 url이 변경되더라도 계속하여 회원가입 요청시의 body값이 넘어오게된다.
이를 위해서 HTTP_302를 호출하여 리다이렉트를 해줄필요가 있다는것을 알았다.
반대로 GET요청이었다면 그냥 302호출없이 리다이렉트가 가능했을것 같다는 생각이 들었다.
### 요구사항 5 - cookie

```java
else if (url.equals("/user/login")){
                String queryString = util.IOUtils.readData(bufferedReader, cl);
                Map<String, String> param = HttpRequestUtils.parseQueryString(queryString);
                User user = DataBase.findUserById(param.get("userId"));
                if( user == null){
                    responseResource(out,"/user/login_failed.html");
                    return;
                }
                if(user.getPassword().equals(param.get("password"))){
                    DataOutputStream dos = new DataOutputStream(out);
                    response302login(dos);
                }else{
                    responseResource(out,"/user/login_failed.html");
                }
            }
...
private void responseResource(OutputStream out, String url) throws IOException {
        try {
            byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
            DataOutputStream dos = new DataOutputStream(out);
            response200Header(dos, body.length);
            responseBody(dos, body);
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
    private void response302login(DataOutputStream dos) {
        try {
            dos.writeBytes("HTTP/1.1 302 Redirect \r\n");
            dos.writeBytes("Set-Cookie: logined=true \r\n");
            dos.writeBytes("Location: /index.html \r\n");
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
         
```
앞에서 했던 과제들의 내용을 활용하여 진행해야 했던 과제였다.
실질적으로 구현해야 했던 부분은 header를 response해주는 부분이었다.
로그인이 실패했을경우에는 http200으로 response해주고 login_failed.html로 redirect해주었다.
성공했을시에는 http302로 response후 Set-Cookie값을 true로 바꿔주고 index.html로 redirect해주었다.

-------------------------------------------------
cookie값의 확인을 통해서 현재 로그인된 계정을 받아오는 과제를 받고나서 서버에서 직접적으로 Html파일소스를 건들수 있다는 것을 처음으로 알게되었다.
```java
else if(url.equals("/user/list")){
                   if(logined!=true){
                       responseResource(out,"/user/login.html");
                       return;
                   }
                   Collection<User> users = DataBase.findAll();
                   StringBuilder sb = new StringBuilder();
                   sb.append("<table border='1'>");
                   for (User user: users){
                       sb.append("<tr>");
                       sb.append("<td>"+user.getUserId()+"</td>");
                       sb.append("<td>"+user.getName()+"</td>");
                       sb.append("<td>"+user.getEmail()+"</td>");
                       sb.append("</tr>");
                   }
                   sb.append("</table>");
                   body = sb.toString().getBytes();
                   DataOutputStream dos = new DataOutputStream(out);
                   response200Header(dos, body.length);
                   responseBody(dos, body);
            }
```
버퍼리더로 부터 읽어온값으로부터 cookie접두사를 찾아서 로그인쿠키값의 확인을 통해서 로그인유무를 판단하였다.
로그인 여부를 판단한후 /user/list헤더로 접속하게 되면 비로그인시 login.html로 리다이렉트 시키고 로그인된 계정이면 html로 StringBuilder를 통해서 엔티티의 get메소드를 통해서 사용자 정보를 읽어온후 코드를 전송해준다.
### 요구사항 6 - stylesheet 적용
```java
else if(url.endsWith(".css")){
                body = Files.readAllBytes(new File("./webapp" + url).toPath());
                DataOutputStream dos = new DataOutputStream(out);
                response200cssHeader(dos, body.length);
                responseBody(dos, body);
            }
```
css 스타일시트 적용은 앞의 과제들에 비하면 상당히 가벼운 과제였다.
단순히 http response값만 text/html -> text/css로만 바꿔주면 알아서 css파일을 적용시켜주었다.

### heroku 서버에 배포 후
* 
