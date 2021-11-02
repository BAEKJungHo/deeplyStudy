# HTTP 요청 데이터 - API 메시지 바디 JSON

JSON 형식으로 데이터를 전송 하고 받기 위해서는 HttpServletRequest 에 담겨있는 JSON 데이터를 객체로 변환(그 반대도 포함)시켜주기 위해
`ObjectMapper`라는 클래스를 이용해야 한다. 스프링 부트의 경우에는 jackson 라이브러리가 자동으로 추가가되어있어서 따로 의존성 설정이 필요 없지만,
스프링의 경우에는 필요하다.

예를 들어 `{"username":"bjh", "age":28}` 다음과 같은 JSON 형식의 데이터를 서버로 전달한다고 했을 때 서버에서 해당 JSON 데이터를 객체로 변환 시키는 과정은 다음과 같다.

1. 메시지 바디의 내용을 bytecode 로 얻기
2. bytecode to String
3. String to Object

## 서블릿을 사용한 코드

```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 메시지 바디 내용을 bytecode 로 얻기
        ServletInputStream inputStream = request.getInputStream();

        // 2. bytecode to String
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); // {"username":"bjh", "age":28} 형식의 문자열
        System.out.println("messageBody = " + messageBody);

        // 3. String to Object
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class); 
        System.out.println("helloData.username = " + helloData.getUsername());
        System.out.println("helloData.age = " + helloData.getAge());

        response.getWriter().write("ok");
    }

}
```

여기서 주의할 점은 String to Object 를 하기 위해 해당 객체에는 Getter 와 Setter 가 존재해야 한다.
Jackon2ObjectMapperBuilder 가 autoDetectGettersSetters() 메서드를 이용해서 String 형식으로된 JSON 을 객체의 속성에 바인딩 시키기 때문이다.
