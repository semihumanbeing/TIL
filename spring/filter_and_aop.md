# API 로그 저장하기 - Filter와 AOP를 사용한

Date: August 24, 2023

### 이야기를 시작하기에 앞서 프로젝트에 대한 설명

내가 만들고 있는 것은 웹과 여러 기관 간의 API 통신을 도와주고, IP로 인증처리를 하고, 로그를 저장하는 API 게이트웨이 같은 부분이다.

### 요구사항

나의 API 게이트웨이를 통해 웹에서도 요청을 하고, 각종 기관들에서도 내 API를 통해 데이터를 입력해 주는데, 이를 전부 로그로 저장하기에는 불필요하게 길고 로그에 저장하면 안되는 민감한 데이터들이 많이 있었다. 또 단순히 내 API를 사용하는 부분에 대한 로그가 아니라 내가 다른 API들에 요청을 보내는 부분에서도 로그를 저장해야 했다.

그러나 만약 프로젝트의 모든 부분에 로그를 넣는 코드를 작성한다면 API에서 외부 연결하는 부분만 그 당시 16군데 정도 되었고 더 늘어날 가능성도 있었기 때문에 Filter와 AOP를 사용하여 로그를 저장하기로 했다.

1. 내 API에 들어오는 요청과 내가 보낸 응답을 필요한 부분만 추출 (Filter)
2. 내 API에서 다른 API에 보낸 요청과 내가 받은 응답에서 필요한 부분만 추출 (AOP)

## 0. Settings

```java
@RestController
@RequestMapping("/api")
public class ApiController {

    @PostMapping("/test")
    public ResponseEntity<?> test(@RequestBody TestVO testVO){
        return ResponseEntity.status(HttpStatus.OK).body(testVO);
    }
}
```

예제를 보여주기 위해 간단한 테스트 컨트롤러를 만들었다. (Java 8, Spring boot 2.7)

```java
@Builder
@Data
@AllArgsConstructor
public class TestVO {
    private String name;
    private String phone;
    private String secret;
}
```

그리고 데이터 가공을 위해 이름, 전화번호, 비밀이 담긴 VO 를 만들었다.

![Untitled](API%20%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20-%20Filter%E1%84%8B%E1%85%AA%20AOP%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20e149bc53e88847b889f829e2c65e1622/Untitled.png)

위와 같은 requestBody 에 친구에 대한 정보를 담아 요청을 보내면 아래와 같은 응답이 돌아온다.

## 1. Filter

요청이 controller에 들어오기 전에 필터에서 들어오는 데이터를 가공하여 로그 데이터를 저장하기 위해 우선 ServletRequest와 ServletResponse의 데이터를 가져와서 사용하려고 했다. 

그런데 request에서 inputstream과 outputstream으로 데이터를 한 번 꺼내면 휘발되기 때문에 막상 controller에서는 request 의 내용을 사용할 수 없어진다! request, response를 controller에 정상적으로 보내고 응답을 받기 위해서는 ServletRequest와 ServletResponse의 데이터를 랩에 싸서 필터에서 날아가지 않게 해 주어야 한다. 

### 1-1 Custom Wrappers

우선 ServletInputStream 으로 들어온 ByteArray 데이터를 InputStream으로 꺼내지 않고 새로운 ByteArrayInputStream에 넣어주는 CachedByteArrayInputStream 을 구현한다.

```java
public class CachedByteArrayInputStream extends ServletInputStream {
    private ByteArrayInputStream inputStream;

    public CachedByteArrayInputStream(byte[] body) {
        this.inputStream = new ByteArrayInputStream(body);
    }

    @Override
    public int read() {
        return inputStream.read();
    }

    @Override
    public boolean isFinished() {
        return false;
    }

    @Override
    public boolean isReady() {
        return false;
    }

    @Override
    public void setReadListener(ReadListener readListener) {

    }
}
```

OutputStream도 만들어준다. 

```java
public class CachedOutputStream extends ServletOutputStream {
    private final DataOutputStream outputStream;

    public CachedOutputStream(OutputStream outputStream) {
        this.outputStream = new DataOutputStream(outputStream);
    }

    @Override
    public void write(int b) throws IOException {
        outputStream.write(b);
    }
    @Override
    public boolean isReady() {
        return false;
    }

    @Override
    public void setWriteListener(WriteListener writeListener) {

    }

}
```

이제 이렇게 캐시된 데이터를 꺼내 사용하기 위한 커스텀 포장지를 만든다.

```java
public class CustomRequestWrapper extends HttpServletRequestWrapper {
    public byte[] body;

    public CustomRequestWrapper(HttpServletRequest request) {
        super(request);
        try {
            DataInputStream dis = new DataInputStream(request.getInputStream());
            ByteArrayOutputStream os = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            for (int length = dis.read(buffer); length != -1; length = dis.read(buffer)){
                os.write(buffer, 0, length);
            }
            os.flush();
            this.body = os.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        return new CachedByteArrayInputStream(this.body);
    }

    public byte[] getBody() { return body; }
    public void setBody(byte[] body) { this.body = body; }
}
```

HttpServletRequestWrapper를 상속받아 getInputStream 했을 때 아까 만든 캐시된 데이터를 가지고 올 수 있도록 메서드를 바꿔치기 해준다.

그리고 그 데이터를 어떻게 가져올지 (getBody .. ) 는 원하는대로 만들면 된다.

```java
public class CustomResponseWrapper extends HttpServletResponseWrapper {
    ByteArrayOutputStream byteArrayOutputStream;
    CachedOutputStream cachedOutputStream;

    public CustomResponseWrapper(HttpServletResponse response) {
        super(response);
        this.byteArrayOutputStream = new ByteArrayOutputStream();
    }

    @Override
    public ServletOutputStream getOutputStream() throws IOException {
        if(cachedOutputStream == null) {
            cachedOutputStream = new CachedOutputStream(byteArrayOutputStream);
        }
        return cachedOutputStream;
    }
    public String getBody() {
        return byteArrayOutputStream.toString(StandardCharsets.UTF_8);
    }
}
```

ResponseWrapper로 만들어준다.

### 1-2 필터에 적용하기

```java
@Component
public class LogFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // 처음 서버에 실행될 때
    }

    @Override
    public void doFilter(ServletRequest servletRequest, 
                         ServletResponse servletResponse, 
                         FilterChain filterChain) throws IOException, ServletException {
        
    }

    @Override
    public void destroy() {
        // 프로세스를 종료할 때
    }
}
```

javax.servlet 의 Filter를 구현한다.

init과 destroy에는 할 것이 없으니 비워주고 doFilter에서 request와 response를 가지고 장난질을 해보자. 

- request와 response의 body를 가지고 올 때 Wrapper 를 사용하면 된다.
- filterchain.doFilter() 메서드를 기준으로 request와 response 시점을 나눌 수 있다.

```java
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        CustomRequestWrapper customRequestWrapper = new CustomRequestWrapper((HttpServletRequest) servletRequest);
        CustomResponseWrapper customResponseWrapper = new CustomResponseWrapper((HttpServletResponse) servletResponse);

        try {
            // request 처리
            String requestLogData = getLogDataFromRequest(customRequestWrapper.getBody());
            System.out.println("request 로그에 저장할 데이터: " + requestLogData);

            // 이곳을 기점으로 
            filterChain.doFilter(customRequestWrapper, customResponseWrapper);
						
						// response 처리
            String body = customResponseWrapper.getBody();
            String responseLogData = getLogDataFromResponse(body);
            System.out.println("response 로그에 저장할 데이터: " + responseLogData);

						// 응답 반환하는 부분
            byte[] responseMessage = body.getBytes(StandardCharsets.UTF_8);
            int length = responseMessage.length;
            servletResponse.setContentLength(length);
            servletResponse.getOutputStream().write(responseMessage);
            servletResponse.flushBuffer();
        } catch (Exception e) {
            int status = customResponseWrapper.getStatus();
            System.out.println("에러 로그 처리 - 에러코드: "+ status);
            throw e;
        } finally {
            customRequestWrapper.setBody(null);
        }

    }
```

데이터를 직접 넣지는 않고 추출만 해보았다.

에러가 날 때에도 ResponseWrapper 의 데이터를 가지고 로그 저장을 할 수 있다.

Response를 보낼 때에는 직접 반환을 해주어야 응답이 돌아온다.

에러를 보낼 때도 마찬가지이다.

```java
    private String getLogDataFromRequest(byte[] body) throws JSONException {
        String data = new String(body, StandardCharsets.UTF_8);
        if(data.startsWith("{")){
            JSONObject json = new JSONObject(data);
            if(json.has("secret")) json.remove("secret");
            return json.toString();
        }
        return data;
    }

    private String getLogDataFromResponse(String body) throws JSONException {
        if(body.startsWith("{")){
            JSONObject json = new JSONObject(body);
            if(json.has("secret")) json.remove("secret");
            if(json.has("phone")) json.remove("phone");
            return json.toString();
        }
        return body;
    }
```

예제를 위해 DB에 저장하지는 않고 콘솔 출력할 데이터를 가져오도록 구현했다.

정책으로 request에서는 json 데이터에 비밀을 로그에 저장하지 않기로 하고,

response에서는 전화번호도 저장하지 않기로 했다.

이렇게 적용한 뒤 아까 사용한 API를 그대로 사용하면 

![Untitled](API%20%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20-%20Filter%E1%84%8B%E1%85%AA%20AOP%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20e149bc53e88847b889f829e2c65e1622/Untitled%201.png)

request, response 시 이런식으로 각각 다른 데이터를 담아 로그를 저장할 수 있다.

controller에서는 어떤 코드도 추가하지 않아도 된다.

이렇게 내 API로 들어오는 모든 요청과 응답에 대한 로그를 남길수 있게 되었다.

이제 내 API에서 외부 API로 보내고 받은 응답을 로그에 남겨보자

## 2. AOP

테스트용으로 testVO로 메시지가 들어오면 외부 API에 요청을날리고, 

그 응답을 맵에 담아 보여주는 예제코드를 작성했다.

```java
@PostMapping("/test2")
public ResponseEntity<?> test2(@RequestBody TestVO testVO) {
    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.setContentType(MediaType.APPLICATION_JSON);
    Map<String, Object> map = restTemplateService.sendRequest(
																	 "http://localhost:8081/api/to-rest-template", 
                                    testVO, 
                                    httpHeaders);
    return ResponseEntity.status(HttpStatus.OK).body(map);
}
```

RestTemplate로 요청을 보내는 기능은 여러곳에서 사용할 수 있도록, 또한 AOP의 조인포인트로 지정하기 용이하도록 sendRequest 라는 이름의 서비스로 만들었다.

```java
@Service
public class RestTemplateServiceImpl implements RestTemplateService {

    @Override
    public Map<String, Object> sendRequest(String url, TestVO testVO, HttpHeaders httpHeaders) throws JsonProcessingException {
        HttpEntity<?> requestEntity = new HttpEntity<>(testVO, httpHeaders);
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.getMessageConverters().add(0, new StringHttpMessageConverter(StandardCharsets.UTF_8));

        ResponseEntity<byte[]> responseEntity = restTemplate.exchange(url, HttpMethod.POST, requestEntity, byte[].class);
        if(responseEntity.getBody() == null) {
            throw new RuntimeException("error!");
        }
        byte[] body = responseEntity.getBody();
        ObjectMapper objectMapper = new ObjectMapper();
        return objectMapper.readValue(new String(body), new TypeReference<Map<String, Object>>() {});
    }
}
```

HttpEntity를 만들어서 헤더와 VO를 포장한다.

`restTemplate.exchange()` 부분에서 객체를 byte array로 변환하여 통신하고 byte[] 형태의 responseEntity 를 받는다.

그 다음 만약 getBody() 한 결과가 있으면 Map으로 변환하여 리턴한다.

그리고 이 모듈이 아닌 8081 포트를 사용하는 다른 스프링부트 프로젝트를 만들어서 test 컨트롤러 2 를 만들었다.

```java
@RestController
@RequestMapping("/api")
public class AnotherApiController {

    @PostMapping("/to-rest-template")
    public ResponseEntity<?> apiToUse(@RequestBody TestVO testVO){
        return ResponseEntity.status(HttpStatus.OK)
                .body(ResultVO.builder()
                        .status("200")
                        .message("your name is... " +testVO.getName())
                        .secret("wooooow").build());
    }
}
```

![Untitled](API%20%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20-%20Filter%E1%84%8B%E1%85%AA%20AOP%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20e149bc53e88847b889f829e2c65e1622/Untitled%202.png)

/api/test2 를 사용하면 8081의 api에 요청을 보내고 받은 응답을 보여준다.

만약 API 수가 얼마 되지 않고 늘어날 일이 없다면 로그를 지정해 줄 수 있겠다. 하지만 만약 API가 20개 30개 되는데 모든 RestTemplate 요청 전후로 비밀을 제외하도록 처리해야 한다면 정말 귀찮을 것이다..

이런식으로 여러 API들에서 공통적으로 시행해야 하는 공통적인 부분을 횡단 관심사 (crosscutting concern) 라고한다. 이런 횡단 관심사를 모듈화하여 유지보수를 쉽게 하기 위해서 AOP를 사용할 수 있다. 나는 AspectJ를 사용했다.

내가 보낸 요청과 8081의 api에서 보낸 응답을 로그에 저장하기 위해 AOP를 사용해보자.

```java
@SpringBootApplication
@EnableAspectJAutoProxy
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
```

우선 @EnableAspectJAutoProxy 를 써서 AspectJ 사용 설정한다.

```java
@Aspect
@Component
public class LogAspect {

    @Before("execution(* com.example.demo.service.RestTemplateService.sendRequest(..)) && args(url, ..)")
    public void beforeApi(JoinPoint joinPoint, String url) throws JsonProcessingException {
        Object[] args = joinPoint.getArgs();
        ObjectMapper objectMapper = new ObjectMapper();
        System.out.println(Arrays.toString(args));
        String s = objectMapper.writeValueAsString(args[1]);
        System.out.println(s);
    }
```

Aspect 를 사용할 클래스를 만들고 @Aspect와 @Component 어노테이션을 설정해 준다.

API를 보내기 전 실행되는 beforeApi 메서드를 만들었다. @Before 어노테이션을 사용한다. 

`@Before("execution(* com.example.demo.service.RestTemplateService.sendRequest(..))`

이렇게 괄호안에 들어간 것은 매핑 패턴이다. 

- execution: 메소드 실행 결합점(join points)과 일치시키는데 사용된다.
- within: 특정 타입에 속하는 결합점을 정의한다.
- this: 빈 참조가 주어진 타입의 인스턴스를 갖는 결합점을 정의한다.
- target: 대상 객체가 주어진 타입을 갖는 결합점을 정의한다.
- args: 인자가 주어진 타입의 인스턴스인 결합점을 정의한다.
- @target: 수행중인 객체의 클래스가 주어진 타입의 어노테이션을 갖는 결합점을 정의한다.
- @args: 전달된 인자의 런타입 타입이 주어진 타입의 어노테이션을 갖는 결합점을 정의한다.
- @within: 주어진 어노테이션을 갖는 타입 내 결합점을 정의한다.
- @annotation: 결합점의 대상 객체가 주어진 어노테이션을 갖는 결합점을 정의한다.

그리고

“*” 는 모든것, “..” 는 여러가지를 의미한다.

따라서 위에 설정된 조인포인트는 “여러 변수들이 있을 수 있는 sendRequest() 라는 이름을 가진 모든 메소드가 실행될때” 를 의미한다.  

그래서 만약 sendRequest(String a) 와 sendRequest(Integer b) 와 같이 사용해도 이 모든 메소드를 실행하기 전에 befroeApi 메소드를 실행시킬 수 있다.

`&& args(url, ..)")` 

args(url, ..)는 첫 번째 인자로 url이라는 변수를 받고, 나머지 인자들은 무시한다. 그리고 이곳에서 선언한 변수는

`beforeApi(JoinPoint joinPoint, String url)` 

위와 같이 메서드에 인자로 전달받아 사용할 수 있다.

오브젝트 배열 args에는 sendRequest 메소드의 객체들이 순서대로 저장되어있다. 그래서 만약

`sendRequest(String url, TestVO testVO, HttpHeaders httpHeaders)`

의 args를 출력하면 아래와 같은 결과가 나온다.

```java
[http://localhost:8081/api/to-rest-template, 
	TestVO(name=김친구, phone=010-1111-2222, secret=비밀이야), 
	[Content-Type:"application/json"]]
```

로그를 처리해야한다면 이 결과를 이용할 수 있겠다.

@After 어노테이션 도 있지만 정상 응답 값과 에러가 났을 때 다른 값을 다룰 때 좀 더 섬세하게 사용할 수 있는 방법을 찾다가 @AfterReturning 이랑 @AfterThrowing 을 알게되어 적용하였다.

```java
@Before("execution(* com.example.demo.service.RestTemplateService.sendRequest(..)) && args(url, ..)")
public void beforeApi(JoinPoint joinPoint, String url) {
    Object[] args = joinPoint.getArgs();
    System.out.println("Before API: " + Arrays.toString(args));
}

@AfterReturning(value = "execution(* com.example.demo.service.RestTemplateService.sendRequest(..))", returning = "response")
public void afterApi(JoinPoint joinPoint, Map<String, Object> response) {
    System.out.println("After API: " + response);
}

@AfterThrowing(value = "execution(* com.example.demo.service.RestTemplateService.sendRequest(..))", throwing = "exception")
public void afterApi(JoinPoint joinPoint, Exception exception) {
    System.out.println("Error Response: " + exception.getMessage());
}
```

각각 매핑 이후에 returning = 과 throwing = 이 있고 이곳에서 지정한 객체를 인자로 넣어주면

@AfterReturning 의 response는 연결된 API에서 return 한 값을 사용할 수 있게 되고

@AfterThrowing 의 exception 은 Exception 의 값을 사용할 수 있게 된다.

위와 같이 간단히 출력만 하는 방식으로 만들었다. 

이제 /api/test2 를 사용하면

![Untitled](API%20%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20-%20Filter%E1%84%8B%E1%85%AA%20AOP%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20e149bc53e88847b889f829e2c65e1622/Untitled%203.png)

Controller나 Service에 어떤 변동도 없이 

1. 내 API에 들어온 요청 값
2. 외부 API에 RestTemplate에 보내는 데이터
3. 외부 API에서 전달받은 Map<String, Object> 형태의 API 응답 데이터
4. 내 API에서 응답한 값

4가지 로그 데이터를 추출할 수 있게 되었다. 

또한 에러를 일부러 발생 시켰을 때에도 (name 이 null이면 에러를 반환하게 했다)

![Untitled](API%20%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20-%20Filter%E1%84%8B%E1%85%AA%20AOP%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20e149bc53e88847b889f829e2c65e1622/Untitled%204.png)

![Untitled](API%20%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20-%20Filter%E1%84%8B%E1%85%AA%20AOP%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20e149bc53e88847b889f829e2c65e1622/Untitled%205.png)

에러로 외부 API에서 받은 응답과 내가 response로 보낸 데이터까지 저장할 수 있다.

이렇게 내 API의 요청과 응답, 외부 API에서의 요청과 응답을 Filter와 AOP를 통해 추출하는 방법에 대해 알아보았다. 간단하게 예제로 작성했지만 처음에 적용할때에는 AOP 매핑이 API에 연결조차 안돼서 머리를 싸매고 고민했던 기억이 있다… 무튼 이 예제를 보고 다음에도 사용할 수 있길..

그 밖에 삽질 기억

- Joinpoint 를 메서드에서 사용 시 org.aspectj.lang.Joinpoint 를 import 할것
- aspect 메서드들은 전부 void 이고 @Around 를 사용하면 ProceedingJoinpoint 를 사용해야한다. 해당 joinpoint를 proceed() 한 전후로 메서드 실행시점이 나뉜다. 그런데 나는 이걸 사용하는 방법은 다 실패하고 API 연결조차 되지 않았고 .. 응답과 에러를 다르게 다루어야 했기 때문에 AfterReturning 과 AfterThrowing을 사용했다.
- 이도 저도 안된다면 LogAspect 를 main 클래스에서 빈 등록한다.