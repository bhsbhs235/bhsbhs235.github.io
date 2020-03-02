---
layout: post
title: "spring-security"
subtitle: "Login,Signup,Forgotpassword with springsecurity"
date: 2020-02-23
background: '/img/posts/bg-img/35.jpg'
comments: true
categories: Springboot
---

<h1 class="section-heading2">서론</h1>

spring boot, spring security 프레임워크를 사용해서 jsp로 로그인, 회원가입, 토큰을 이용한 비밀번호 변경, 권한에 따른 메뉴 접근 권한을 구현해 보았습니다. 

최대한 한 줄 한 줄 자세하게 적도록 노력하였습니다 **의미**를 생각하시면서 봐주시면 적용하기 쉬울 것으로 예상됩니다.

가장 기본이 되는 것은 공식 문서입니다. 제 글은 참고용일 뿐이며, 자세한 내용은 해당 프레임워크의 공식문서를 참고해주세요.

<h1 class="section-heading2">본문</h1>

| 개발환경 | 비고 |
|---|:---:|
| 프레임워크 | spring-boot,spring-security(2.2.4.RELEASE)|
| DB | MYSQL(JPA/Hibernate) |

#### 주요화면

##### Main

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot2.JPG">	
</div>

##### Login

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot4.JPG">	
</div>

##### Sign up

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot3.JPG">	
</div>

##### Forgot password

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot5.JPG">	
</div>

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot6.JPG">	
</div>

#### build.gradle

``` Groovy
plugins {
	id 'org.springframework.boot' version '2.2.4.RELEASE'
	id 'io.spring.dependency-management' version '1.0.9.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'
targetCompatibility = '1.8'

configurations {
	developmentOnly
	runtimeClasspath {
		extendsFrom developmentOnly
	}
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	compile 'org.apache.tomcat.embed:tomcat-embed-jasper' // 내장 톰캣
	compile 'javax.servlet:jstl:1.2' // Jsp Standard Tag Library
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa' // JPA
	implementation 'org.springframework.boot:spring-boot-starter-security' // spring-security
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-mail' // 토큰을 보낼 mail library
	implementation 'org.springframework.security:spring-security-taglibs' // security tag library
	implementation 'org.modelmapper:modelmapper:2.3.5' // Entity를 Dto로 바꿔줄 library
	compileOnly 'org.projectlombok:lombok' // 애노테이션
    annotationProcessor 'org.projectlombok:lombok' // 애노테이션
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	runtimeOnly 'mysql:mysql-connector-java' // mysql connector
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
	testImplementation 'org.springframework.security:spring-security-test'
}

test {
	useJUnitPlatform()
}

```

필요한 의존 라이브러리를 추가합니다.

#### JPA and Hibernate

JPA는 별도의 구현 클래스 없이 인터페이스만을 사용할 수 있도록 제공합니다. 제공되는 인터페이스 JpaRepository는 실행시점에 자동으로 인터페이스 내용을 연결하는 엔티티에 맞게 자동으로 구현해줍니다. 만약 스프링 JPA 인터페이스에서 제공하지 않는 기능을 사용하고 싶을 때는 메서드명을 특정한 규칙대로 만들어서 사용하면 인터페이스가 알아서 그 이름에 맞는 JPQL을 만들어서 실행해줍니다. 자세한 내용 설명은 생략하도록 하겠습니다.

##### Entities

DB의 엔티티를 정의

###### MemberEntity.java

``` Java
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 기본 생성자 생성
@Getter // getter 생성 
@Entity // Entity 지정
@DynamicUpdate // update시 변경된 컬럼만 변경
@Table(name = "member") // 해당 이름으로 table생성
public class MemberEntity {
    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    @Column(name = "id", nullable = false, updatable = false)
    private Long id;

    @Column(length = 15, nullable = false, unique = true)
    private String username;

    @Column(length = 100, nullable = false)
    private String password;

    @Transient
    private String passwordConfirm;

    @Column(nullable = false, unique = true)
    @Email(message = "please provide a valid e-mail")
    @NotEmpty(message = "please provide an e-mail")
    private String email;

    @Column(name = "reset_token")
    private String resetToken;

    @Builder
    public MemberEntity(Long id, String username, String password, String email, String resetToken) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.email = email;
        this.resetToken = resetToken;
    }
}
```

유저를 Entity로 생성해줍니다 

Hibernate에서 위의 내용대로 Table을 생성해줍니다.

> **Note :** Entity에 Setter를 두지 않는 이유는 setter 메소드를 통해 값을 수정할 수 있기 때문에 보안상 문제가 생길 수도 있기 때문입니다. Entity는 DB에 직접적인 영향을 주기 때문에 값을 수정하는 것에 민감합니다. 하지만 Dto는 Controller,Service 등 내부에서 단순히 정보를 주고 받기 위함이기 때문에 수정할 수 있어야 하며 따라서 Setter,Getter 모두 둡니다

###### Role.java

``` Java
@AllArgsConstructor // 모든 필드값을 파라미터로 하는 생성자 생성
@Getter // getter 생성
public enum Role {
    ADMIN("ROLE_ADMIN"),  // admin계정 권한
    MEMBER("ROLE_MEMBER");// 일반 유저 권한

    private String value;
}
```

##### JpaRepository

데이터를 쉽고 빠르게 검색할 수 있도록 SQL을 자동생성 해주는 인터페이스

###### MemberRepository.java

``` Java
public interface MemberRepository extends JpaRepository<MemberEntity, Long>{
    Optional<MemberEntity> findByUsername(String username);
    Optional<MemberEntity> findByEmail(String email);
    Optional<MemberEntity> findByResetToken(String resetToken);
}
```

각 컬럼 값을 바탕으로 유저의 정보를 가져옵니다. (select)

##### MYSQL JDBC

MYSQL DB서버와 DataSource로 connection 및 등록한다. 또한, JPA 설정을 기술합니다.

###### application.properties

``` default
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/user?characterEncoding=UTF-8&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.database-platform=org.hibernate.dialect.MySQL57Dialect
spring.jpa.generate-ddl=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true
```

Mysql 서버와 연결해줍니다 localhost 포트번호 3306번으로 연결했습니다.

sql 서버 username과 password를 입력하고

spring.jpa.database-platform - JPA 데이터베이스 플랫폼을 지정합니다.

spring.jpa.show-sql - 콘솔에 JPA 실행 쿼리를 출력합니다.

spring.jpa.properties.hibernate.format_sql - 콘솔에 출력되는 JPA 실행 쿼리를 가독성있게 표현합니다.

spring.jpa.hibernate.ddl_auto - 데이터베이스 초기화 전략을 설정합니다.

none - 아무것도 실행하지 않습니다.

create - SessionFactory가 시작될 때 기존테이블을 삭제 후 다시 생성합니다.

create-drop - create와 같으나 SessionFactory가 종료될 때 drop을 실행합니다.

update - 변경된 스키마만 반영합니다.

validate - 엔티티와 테이블이 정상적으로 매핑되었는지만 확인합니다.

#### Spring security 설정

##### UserDetails

spring security에서 제공하는 사용자의 **인증(Authentication)**정보 담아두는 인터페이스 입니다.

###### UserDetailsServiceImpl.java

``` Java
@Service
public class UserDetailsServiceImpl implements UserDetailsService{
    @Autowired
    private MemberRepository memberRepository;

    @Override
    @Transactional(readOnly = true) // DB에서 읽기 전용으로 설정 (insert,update,delete예외발생)
    public UserDetails loadUserByUsername(String username) {
        Optional<MemberEntity> memberEntity = memberRepository.findByUsername(username);
        if (!memberEntity.isPresent()) throw new UsernameNotFoundException(username);

        Set<GrantedAuthority> grantedAuthorities = new HashSet<>();

        if (("admin123").equals(username) || ("test123".equals(username))) {
            grantedAuthorities.add(new SimpleGrantedAuthority(Role.ADMIN.getValue()));
        } else {
            grantedAuthorities.add(new SimpleGrantedAuthority(Role.MEMBER.getValue()));
        }

        return new org.springframework.security.core.userdetails.User(memberEntity.get().getUsername(), memberEntity.get().getPassword(), grantedAuthorities);
    }
}
```

로그인시 username(id)가 "admin123"일 경우 "ADMIN" 권한을 부여하고 나머지는 기본 유저인 "MEMBER" 권한을 부여합니다

권한,인증 내용을 UserDetails에 보관해두어, 갖고 있는 권한에 따라서 해당 메뉴에 접근할 수 있습니다.

##### Security Configuration

spring security 관련 설정

###### WebSecurityConfiguration.java

``` Java
@Configuration
@EnableWebSecurity
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Qualifier("userDetailsServiceImpl")
    @Autowired
    private UserDetailsService userDetailsService;

    @Bean // spring-security에서 제공하는 암호화방식
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    
    @Override
    public void configure(WebSecurity web) throws Exception
    {
        // static 디렉터리의 하위 파일 목록은 인증 무시 ( = 항상통과 )
        web.ignoring().antMatchers("resources/css/**", "resources/js/**", "resources/img/**", "resources/lib/**");
    }
    
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/admin/**").hasRole("ADMIN") // /admin/으로 시작하는 URL은 "ADMIN"권한이 있는 유저만 접근가능
                .antMatchers("/member/**").hasRole("MEMBER") // /member/으로 시작하는 URL은 "MEMBER"권한이 있는 유저만 접근가능
                .antMatchers("/**").permitAll() // 나머지는 모두가 접근가능 
            .and()
                .formLogin()
                .loginPage("/login") 
                .usernameParameter("username")
                .passwordParameter("password")
                .defaultSuccessUrl("/")
                .permitAll()
            .and()
                .logout()
                .logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/login")
                .invalidateHttpSession(true) 
            .and()
                .exceptionHandling().accessDeniedPage("/403error");
    }

    @Bean
    public AuthenticationManager customAuthenticationManager() throws Exception {
        return authenticationManager();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
    }
}

``` 

``` Java
        antMatchers("/admin/**").hasRole("ADMIN") 
        .antMatchers("/member/**").hasRole("MEMBER") 
        .antMatchers("/**").permitAll() 
```

/admin/으로 시작하는 URL은 "ADMIN"권한이 있는 유저만 접근가능

/member/으로 시작하는 URL은 "MEMBER"권한이 있는 유저만 접근가능

나머지는 모두가 접근가능합니다. 

``` Java
        antMatchers("/admin/**").hasRole("ADMIN") 
        .antMatchers("/member/**").hasRole("MEMBER") 
        .antMatchers("/auth/**").authenticated() 
        .antMatchers("/**").permitAll() 
```

예제로 하나 더 설명드리면,

authenticated()는 **인증**이 완료된 유저만 접근 가능합니다.

여기서 **인증**을 쉽게 말하면 Login한 주체를 말합니다.

우리가 어느 사이트를 들어갔을 때 로그인을 하지 않은 상태로도 볼 수 있는 상태가 인증되지 않은 상태이고,

login 성공을 하면 인증을 한 상태입니다. 하지만 **권한**이 주어진 것은 아닙니다

그리고 위의 권한들은 인증이 끝난 유저에 관해서 부여되는 것입니다. 즉, 인증이 안되면 권한은 당연히 부여가 되지 않습니다.

따라서 /auth/ 로 시작하는 URL 매핑은 "ADMIN", "MEMBER" 권한 상관없이 로그인한 주체면 모두 접근할 수 있습니다.

그리고 /admin/, /member/, /auth/ 를 제외한 나머지는 모든 유저 (익명의 유저)가 접근 가능합니다.
 
``` Java
        .formLogin()
        .loginPage("/login") 
        .usernameParameter("username") //login.jsp에서 username 받을 변수
        .passwordParameter("password") //login.jsp에서 password 받을 변수
        .defaultSuccessUrl("/")
         .permitAll()
```

loginPage는 **인증**이 필요한 페이지에 접근하려고 할 때 리다이렉팅 되는 URL입니다.

다시말하면 인증이 안된 상태에서 인증이 필요한 페이지에 접근할 때 리다이렉팅 되는 URL입니다

즉, 로그인 페이지로 가는 URL입니다.

또한 모든 주체가 로그인 페이지까지는 접근 가능해야하므로 permitAll()을 추가해줍니다.

``` Java
        .logout()
        .logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
        .logoutSuccessUrl("/login")
        .invalidateHttpSession(true) 
```

logoutSuccessUrl은 logout성공시 리다이렉팅되는 URL입니다.

invalidateHttpSession은 logout후 세션에 담겨있는 유저정보를 지우기 위함입니다.

###### index.jsp

``` Html
        <sec:authorize access="hasRole('ROLE_ADMIN')">
            <li class="nav-item">
                <a class="nav-link" href="/admin/">Admin</a>
            </li>
        </sec:authorize>
            <li class="nav-item active">
              <a class="nav-link" href="#">Home
                <span class="sr-only">(current)</span>
              </a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">About</a>
            </li>
        <sec:authorize access="hasRole('ROLE_MEMBER')">
            <li class="nav-item">
                <a class="nav-link" href="/member/">MyInfo</a>
            </li>
        </sec:authorize>
```

(자세한 내용 생략)

jsp에서 spring security tag library를 사용해 해당 권한을 가지고 있는 유저만 보이게 할 수 있습니다.

#### Service

##### MemberService

유저 정보 검색,삽입 등 유저에 관한 모든 Service

###### MemberServiceImpl.java
``` Java
@Service
public class MemberServiceImpl implements MemberService {
    
    @Autowired
    private MemberRepository memberRepository;

    @Autowired
    private ModelMapper modelMapper;
    
    @Override
    public void joinMember(MemberDto memberDto) {
        //비밀번호 암호화
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        memberDto.setPassword(passwordEncoder.encode(memberDto.getPassword()));

        memberRepository.save(memberDto.toEntity()); // DB에 insert
    }
    
    @Override
    public List<MemberDto> findMemberByEmail(String email){

        Optional<MemberEntity> member = memberRepository.findByEmail(email);
        List<MemberDto> members = new ArrayList<MemberDto>();

        if(member.isPresent()){
            MemberDto memberDto = modelMapper.map(member.get(), MemberDto.class);
            members.add(memberDto);
        }

        return members;
    }

    @Override
    public List<MemberDto> findMemberByResetToken(String resetToken){

        Optional<MemberEntity> member = memberRepository.findByResetToken(resetToken);
        List<MemberDto> members = new ArrayList<MemberDto>();

        if(member.isPresent()){
            MemberDto memberDto = modelMapper.map(member.get(), MemberDto.class);
            members.add(memberDto);
        }

        return members;
    }

    @Override
    public void saveMember(MemberDto memberDto){
        memberRepository.save(memberDto.toEntity()); // DB에 insert
    }
}
```

여기서 중요한 내용은 DB에서 Entity로 가져온 후 Dto로 바꾸고 Controller에 던져주고, 반대로 Controller에서 Dto로 받은 후 Entity로 바꾸고 DB에 다시 적용합니다.

왜 굳이 Entity와 Dto를 구분해서 운영할까요? 언뜻보면 다루는 내용(값)도 비슷하고 용도도 비슷해 보이는데 말이죠

구분해서 운영하는 이유는 또다시 ```보안``` 때문입니다.

Entity에서 모든 컬럼의 데이터를 return할 필요가 없습니다.

사용자의 이름과 성별 정도만 받으면 되는데 Entity를 넘겨버리면 주민번호,주소 등등이 모두 넘어가버리는 것입니다.

그럼 개인정보가 노출될 가능성이 매우 커지는 것입니다.

modelMapper는 Entitiy를 Dto로 바꿔주는 인터페이스입니다.

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot10.JPG">	
</div>

##### ForgotPassword token EmailService

비밀번호 변경을 위한 token발급을 Email로 Service 합니다

###### EmailServiceImpl.java

``` Java
@Service
public class EmailServiceImpl implements EmailService{

    @Autowired
    private JavaMailSender mailSender;

    @Async
    public void sendEmail(SimpleMailMessage email){
        mailSender.send(email);
    }
}
```

springframework에서 제공하는 Java기반의 JavaMailSender을 사용해서 쉽게 Email를 보낼 수 있습니다.

또한 메일서버(SMTP server)를 사용하기 위해 application.properties에 등록해줍니다

저는 Gmail을 사용했습니다

``` default
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=gmailID@gmail.com
spring.mail.password=gmailpasswd
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

또한 어플리케이션을 이용해 Gmail에서 보내려면 보안 수준을 활성화 해주어야 합니다.

[보안 수준이 낮은 앱의 액세스](https://myaccount.google.com/lesssecureapps)에서 사용으로 활성화 해줍니다.

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot11.JPG">	
</div>

###### MainController.java

일부 생략

``` Java
@PostMapping("/forgot")
    public ModelAndView processForgotPassword(ModelAndView modelAndView, @RequestParam("email")String email, HttpServletRequest request, RedirectAttributes redir){

        //lookup user in database by email
        List<MemberDto> member = memberService.findMemberByEmail(email);

        if(member.isEmpty())
        {
           modelAndView.addObject("errorMessage", "We didn't find an account for that e-mail address.");
        }else{

            // Generate random 36-character string token for reset password
            MemberDto memberDto = member.get(0);
            memberDto.setResetToken(UUID.randomUUID().toString());

            // Save token to database
            memberService.saveMember(memberDto);

            String appUrl = request.getScheme() + "://" + request.getServerName() + ":8080";

            //Email message
            SimpleMailMessage passwordResetEmail = new SimpleMailMessage();
            passwordResetEmail.setFrom("bhsbhs235@gmail.com");
            passwordResetEmail.setTo(memberDto.getEmail());
            passwordResetEmail.setSubject("Password Reset Request");
            passwordResetEmail.setText("To reset your password, click the link below: \n" + appUrl + "/reset?token="+memberDto.getResetToken());
            emailService.sendEmail(passwordResetEmail);

            // Add success message to view
            modelAndView.addObject("successMessage", "A password reset link has been sent to" + email);
        }
            modelAndView.setViewName("forgotPassword");
            return modelAndView;
    }
```

UUID로 토큰을 만들어서 메일을 작성해 보냅니다.

<div>
	<img class="img-fluid" src="/img/posts/springboot/springboot8.JPG">	
</div>

그럼 위와 같이 메일이 오는데 링크를 클릭하면 비밀번호 변경창이 뜹니다

#### Validator

회원가입시 조건에 충족하도록 검사기를 둡니다.

###### UserValidator.java

``` Java
@Override
    public void validate(Object o, Errors errors) {
        MemberDto memberDto = (MemberDto) o;
        
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "username", "NotEmpty");
        if (memberDto.getUsername().length() < 6 || memberDto.getUsername().length() > 32) {
            errors.rejectValue("username", "Size.userForm.username");
        }
        
        if (memberRepository.findByUsername(memberDto.getUsername()).isPresent()) {
            errors.rejectValue("username", "Duplicate.userForm.username");
        }
        
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "password", "NotEmpty");
        if (memberDto.getPassword().length() < 8 || memberDto.getPassword().length() > 32) {
            errors.rejectValue("password", "Size.userForm.password");
        }

        if (!memberDto.getPasswordConfirm().equals(memberDto.getPassword())) {
            errors.rejectValue("passwordConfirm", "Diff.userForm.passwordConfirm");
        }
    }
```

MainControoller에서 /signup (회원가입)시 validator에서 해당 조건이 충족하는지 검사합니다.

각 조건에 따라 reject Value값을 보내어 validation.properties에서 값에 따른 내용을 가져옵니다.

###### validation.properties

``` default
NotEmpty=This field is required.
Size.userForm.username=Please use between 6 and 32 characters.
Duplicate.userForm.username=Someone already has that username.
Size.userForm.password=Try one with at least 8 characters.
Diff.userForm.passwordConfirm= These passwords don\'t match.
```

<h1 class="section-heading2">결문</h1>

지금까지 긴 글을 읽어주셔서 감사합니다. 나머지 더 세세한 내용은 양이 너무 많아 생략하게 되었습니다 모르시는 부분이 있으시면 댓글 또는 이메일로 연락주시면 바로 답변해드리겠습니다.

감사합니다.

<h1 class="section-heading2">참고문서</h1>

[springboot-springsecurity](https://github.com/bhsbhs235/springboot-springsecurity) - 해당 프로젝트 Github 주소

[spring security refernce](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/) - Spring Security 공식문서

