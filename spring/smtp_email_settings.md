20230328

Spring Boot와 JavaxMail 라이브러리를 사용하여 메일 보내는 api만들기

1. Dependency 설정하기
```
	<!-- JavaMail API -->
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
            <version>1.6.2</version>
        </dependency>
        <!-- Spring Boot Starter Mail -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

2. application.properties 설정하기
Spring boot로 하면 설정이 다 되어 있기 때문에 환경 변수만 설정 해주면 된다
네이버 메일을 사용한다
```
spring.mail.host=smtp.naver.com
spring.mail.port=587
spring.mail.username=ididid@naver.com
spring.mail.password=비밀번호
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

3. Service 만들기
```java
@Service
public class MailService {

    @Value("${spring.mail.username}")
    private String senderEmail;

    private final JavaMailSender javaMailSender;

    @Autowired
    public MailService(JavaMailSender javaMailSender) {
        this.javaMailSender = javaMailSender;
    }

    public void sendEmail(String receiverEmail) throws Exception {
        String subject = "제목";
        String message = "내용";
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage);
        mimeMessageHelper.setFrom(new InternetAddress(senderEmail));
        mimeMessageHelper.setTo(receiverEmail);
        mimeMessageHelper.setSubject(subject);
        mimeMessageHelper.setText(message);
        javaMailSender.send(mimeMessage);
    }
}
```

이제 컨트롤러에서는 서비스를 사용하기만 하면 된다.
제목과 내용을 환경변수에 설정하면 인코딩이 되지 않으니 값을 어플리케이션 단에서 만들어야한다.

4. Controller 만들기
```
@RestController
public class EmailController {
    private MailService mailService;

    @Autowired
    public EmailController(MailService mailService) {
        this.mailService = mailService;
    }

    /**
     * 이메일 보내기
     */
    @GetMapping("/send")
    public String sendEmail(@RequestParam(name = "email") String email){
        try{
            mailService.sendEmail(email);
            return "email sent successfully";
        }catch (Exception e){
            return "Error sending messages " + e.getMessage();
        }
    }
}
```
