# RestTemplate 基本使用

### 1、服务提供者

```java
@RestController
@RequestMapping("/user/user")
@Slf4j
public class UserController {
    @Value("${server.port}")
    private String serverPort;

    @Autowired
    private final UserService userService;

    // GET 请求，不带参数
    @GetMapping("/all")
    public MessageBox<List<UserEntity>> findAll() {...}

    // GET 请求，路径传参
    @GetMapping("/{id}")
    public MessageBox<UserEntity> findByPrimaryKey(@PathVariable("id") Long id) {...}

    // GET 请求，表单传参
    @GetMapping("/by_name_and_age")
    public MessageBox<List<UserEntity>> findAllByNameAndAge(@RequestParam("name") String name, @RequestParam("age") Integer age) {...}

    // POST 请求，请求体传参
    @PostMapping("/by_name_and_age")
    public MessageBox<List<UserEntity>> findAllByNameAndAge2(@RequestBody UserEntity userEntity) {...}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserEntity implements Serializable {
    private static final long serialVersionUID = -1753663566807763438L;
    private Long id;
    private String name;
    private String sex;
    private Integer age;
}
```

### 2、服务消费者

```java
@RestController
@RequestMapping(path = "/payment/payment")
@Slf4j
public class PaymentController {
    public static final String INVOKE_URL = "http://cloud-user-service";

    private final RestTemplate restTemplate;

    @Autowired
    public PaymentController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    /**
     * xxxForEntity 方法
     */
    @GetMapping("/get_for_entity")
    public void getForEntity() {
        ResponseEntity<MessageBox> responseEntity = restTemplate.getForEntity(INVOKE_URL + "/user/user/all", MessageBox.class);

        HttpHeaders httpHeaders = responseEntity.getHeaders();
        log.info("响应头：{}", JSONUtil.toJsonPrettyStr(httpHeaders));

        int statusCodeValue = responseEntity.getStatusCodeValue();
        log.info("响应码：{}", statusCodeValue);

        MessageBox responseBody = responseEntity.getBody();
        log.info("响应体：{}", JSONUtil.toJsonPrettyStr(responseBody));
    }

    /**
     * xxxForObject 方法
     */
    @GetMapping("/get_for_object")
    public void getForObject() {
        MessageBox responseBody = restTemplate.getForObject(INVOKE_URL + "/user/user/all", MessageBox.class);
        log.info("响应体：{}", JSONUtil.toJsonPrettyStr(responseBody));
    }

    /**
     * 路径传参
     *
     * @param id -
     */
    @GetMapping("/get_with_path_variable/{id}")
    public void getWithPathVariable(@PathVariable Long id) {
        MessageBox responseBody = restTemplate.getForObject(INVOKE_URL + "/user/user/" + id, MessageBox.class);
        log.info("响应体：{}", JSONUtil.toJsonPrettyStr(responseBody));
    }

    /**
     * 数字占位符传参
     *
     * @param name -
     * @param age  -
     */
    @GetMapping("get_with_param1")
    public void getWithParam1(@RequestParam("name") String name, @RequestParam("age") Integer age) {
        MessageBox responseBody = restTemplate.getForObject(INVOKE_URL + "/user/user/by_name_and_age?name={1}&age={2}", MessageBox.class, name, age);
        log.info("响应体：{}", JSONUtil.toJsonPrettyStr(responseBody));
    }

    /**
     * Map 传参
     *
     * @param name -
     * @param age  -
     */
    @GetMapping("get_with_param2")
    public void getWithParam2(@RequestParam("name") String name, @RequestParam("age") Integer age) {
        Map<String, Object> params = new HashMap<>(2);
        params.put("name", name);
        params.put("age", age);

        MessageBox responseBody = restTemplate.getForObject(INVOKE_URL + "/user/user/by_name_and_age?name={name}&age={age}", MessageBox.class, params);
        log.info("响应体：{}", JSONUtil.toJsonPrettyStr(responseBody));
    }

    /**
     * Body 传参
     */
    @PostMapping("post_with_body")
    public void postWithBody(@RequestBody UserEntity userEntity) {
        MessageBox responseBody = restTemplate.postForObject(INVOKE_URL + "/user/user/by_name_and_age", userEntity, MessageBox.class);
        log.info("响应体：{}", JSONUtil.toJsonPrettyStr(responseBody));
    }

    @PostMapping("exchange_with_body")
    public void exchangeWithBody(@RequestBody UserEntity userEntity) {
        HttpEntity<UserEntity> httpEntity = new HttpEntity<>(userEntity);

        ResponseEntity<MessageBox> responseBody = restTemplate.exchange(INVOKE_URL + "/user/user/by_name_and_age", HttpMethod.POST, httpEntity, MessageBox.class);
        log.info("响应体：{}", JSONUtil.toJsonPrettyStr(responseBody));
    }
}
```

