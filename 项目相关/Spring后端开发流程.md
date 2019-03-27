本篇按照惯例，只是说明项目中应该如何进行开发，并不会深入地说明具体的技术。
# 开发
后端在开发中的职责主要是创建相应的接口。为完成这个任务，你需要定义Model、创建Dao、创建Controller、创建Service。下边我们一步步来讲这个流程。  
我们以添加导师增删改查这样一个简单的功能为例。我们演示添加导师这样一个功能的实现。
## 创建Controller
在开始之前，我们需要先讨论好这样一个功能的设计。其中一步就是前后端接口的规定，这点涉及到Controller以及model的编写。    
根据设计，添加导师这个功能的接口规范如下(详见rap上的接口定义)    

**请求类型**  
post  
**请求url**  
api/manage-teacher/add-teacher  
**请求参数**
变量名 | 含义 | 类型 | 备注
---|---|---|---
email | 邮箱 | string |
phoneNumber | 电话 | string |
tutorName | 导师姓名 | string |
tutorTitle | 导师职称 | string |
**响应参数列表**  
无
  
  
根据我们接口的规范，我们需要定义以下Controller，以及包装数据的一个model。这里定义的Controller只是一个外壳，具体的实现需要等到我们把Service编写完成之后再回来补充这部分。
```java
@RestController
//在这里设置url,会为内部的每个方法url加上此前缀
@RequestMapping("/api/manage-teacher")
public class ManageTeacherController {
    
    @PostMapping("add-teacher")
    public void addTutor(@RequestBody TutorDTO dto) {
        
    }
}
```
```java
public class TutorDTO {
    private String tutorName;
    private String tutorTitle;
    private String phoneNumber;
    private String email;

    public String getTutorName() {
        return tutorName;
    }

    public void setTutorName(String tutorName) {
        this.tutorName = tutorName;
    }

    public String getTutorTitle() {
        return tutorTitle;
    }

    public void setTutorTitle(String tutorTitle) {
        this.tutorTitle = tutorTitle;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

## 创建Domain
我们需要根据数据库的设计创建实体。
我们根据数据库中的导师表tutor。

字段名 | 类型
---|---
tutor_id | int
tutor_name | varchar
tutor_title | varchar
phone_number | varchar
email | varchar

以下实体可以用idea添加jpa module后使用persistence生成

```java
@Entity
public class Tutor {
    private int tutorId;
    private String tutorName;
    private String tutorTitle;
    private String phoneNumber;
    private String email;

    @Id
    @Column(name = "tutor_id", nullable = false)
    public int getTutorId() {
        return tutorId;
    }

    public void setTutorId(int tutorId) {
        this.tutorId = tutorId;
    }

    @Basic
    @Column(name = "tutor_name", nullable = true, length = 30)
    public String getTutorName() {
        return tutorName;
    }

    public void setTutorName(String tutorName) {
        this.tutorName = tutorName;
    }

    @Basic
    @Column(name = "tutor_title", nullable = true, length = 20)
    public String getTutorTitle() {
        return tutorTitle;
    }

    public void setTutorTitle(String tutorTitle) {
        this.tutorTitle = tutorTitle;
    }

    @Basic
    @Column(name = "phone_number", nullable = true, length = 20)
    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }

    @Basic
    @Column(name = "email", nullable = true, length = 30)
    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Tutor tutor = (Tutor) o;

        if (tutorId != tutor.tutorId) return false;
        if (tutorName != null ? !tutorName.equals(tutor.tutorName) : tutor.tutorName != null) return false;
        if (tutorTitle != null ? !tutorTitle.equals(tutor.tutorTitle) : tutor.tutorTitle != null) return false;
        if (phoneNumber != null ? !phoneNumber.equals(tutor.phoneNumber) : tutor.phoneNumber != null) return false;
        if (email != null ? !email.equals(tutor.email) : tutor.email != null) return false;

        return true;
    }

    @Override
    public int hashCode() {
        int result = tutorId;
        result = 31 * result + (tutorName != null ? tutorName.hashCode() : 0);
        result = 31 * result + (tutorTitle != null ? tutorTitle.hashCode() : 0);
        result = 31 * result + (phoneNumber != null ? phoneNumber.hashCode() : 0);
        result = 31 * result + (email != null ? email.hashCode() : 0);
        return result;
    }
}
```
## 创建Dao

### 规范
我们项目中使用的Dao框架是Spring Data Jpa。它有一个Repository的概念，一个Repository对应一张数据表。   
#### 一个Repository对应一张数据表
虽然Repository还提供了很多强大的功能，但在我们的项目中，我们限制Repository只能对一张表进行基本的增删改查，不能进行连接。   
这样做的一个原因是为了尽量将业务逻辑放在Service层，提高Dao层的通用性。   
另外一个原因是因为之前的数据库设计不好，比较难在使用连接的情况下还能提供通用的接口。
#### Repository命名规范
为保持一致的命名规范，Repository命名采用**Domain名 + Repository**。  
范例：
> TutorRepository  
  TempTeamRepository
  
### 创建
然后我们创建一个访问数据库的对象。我们用的是Spring Data Jpa，这里就叫做Respository。
```java
@Repository
public interface TutorRepository extends CrudRepository<Tutor,Integer>{
}
```
由于继承了CrudRepository这个接口，它已经带有一些基本的查询，这里我们就不另外进行编写。
## 创建Service
我们主要的业务流程都在Service里边实现。我们做个类比，Controller的方法就类似于java类中的main方法，在main方法里边，我们主要使用Service来完成绝大部分的功能。

下面我们定义一个导师增删改查的Service。需要把前边定义的Repository注入。然后实现对导师的添加功能，将前端传来的信息写到数据库里边。
```java
@Service
public class ManageTutorService {

    private final TutorRepository repository;

    public ManageTutorService(TutorRepository repository) {
        this.repository = repository;
    }

    public void addTutor(TutorDTO dto) {
        Tutor tutor = new Tutor();
        tutor.setEmail(dto.getEmail());
        tutor.setPhoneNumber(dto.getPhoneNumber());
        tutor.setTutorTitle(dto.getTutorTitle());
        tutor.setTutorName(dto.getTutorName());
        
        // 调用repository的save方法将数据写入数据库
        repository.save(tutor);
    }
}
```

## 完成Controller
现在，我们已经完成了Service，可以来完成Controller了。只要把这个Service注入Controller里边，调用其方法即可。
```java
@RestController
@RequestMapping("/api/manage-teacher")
public class ManageTeacherController {

    @Autowired
    private ManageTutorService tutorService;
    
    @PostMapping("add-teacher")
    public void addTutor(@RequestBody TutorDTO dto) {
        
        tutorService.addTutor(dto);
    }
}
```
至此，我们完成了一个业务逻辑的编写，可以开始编写单元测试。

# 单元测试
单元测试此处是指在没有和前端连接的情况下对后端功能进行测试。我们一般需要测试三个部分：Controller,Service,Dao。由于Dao由Spring Data Jpa给我们生成，我们大多数情况可以不用进行测试。
## 测试顺序
测试模块的顺序一般是由小到大，由内而外。比如，Controller调用Service,Service调用dao。我们需要先从最小的dao测试开始，然后再测试service，最后测试controller。
## 测试Service
我们的测试都放在项目的test目录下。  
现在，我们创建一个测试以测试上边定义的service方法。注入我们的service，然后编写一个方法，测试我们的service。
```java

@SpringBootTest // 此注解说明这个test是一个bean
@RunWith(SpringRunner.class) // 此注解指明该test需要启动spring整个项目
public class ManageTeacherServiceTest {

    @Autowired
    private ManageTutorService service;
    
    @Test
    public void testAddTeacher() throws Exception {
        TutorDTO dto = new TutorDTO();
        dto.setEmail("39411@qq.com");
        dto.setPhoneNumber("13390345361");
        dto.setTutorName("黄炜");
        dto.setTutorTitle("教授");
        
        service.addTutor(dto);
    }
}
```
编写完毕后，就可以在idea像运行main方法一样运行这个test了。运行完毕后，查看数据库一下，看是否成功插入数据。
## 测试Controller
测试Controller会相对比较麻烦。我们需要模拟http请求传到我们Controller中的情况。所幸Spring已经提供了这个功能给我们。我们需要用到mockmvc来模拟http请求。

```java
@SpringBootTest
@RunWith(SpringRunner.class)
@AutoConfigureMockMvc // 此注解自动配置mockmvc
public class ManageTeacherControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void testAddTutor() throws Exception {

        // 此对象可以将object转为json字符串
        ObjectMapper mapper = new ObjectMapper();

        TutorDTO dto = new TutorDTO();
        dto.setEmail("39411@qq.com");
        dto.setPhoneNumber("13390345361");
        dto.setTutorName("黄炜");
        dto.setTutorTitle("教授");

        String json = mapper.writeValueAsString(dto);

        // 调用perform以开始伪造请求
        MvcResult result = mvc.perform(
                post("/api/manage-teacher/add-teacher") // 请求url
                        .contentType(MediaType.APPLICATION_JSON) // 请求的类型
                        .characterEncoding("utf-8") // 编码方式
                        .content(json)) // 前台传递过来的时候只是一个json字符串，这里也要模拟前台
                .andDo(print()) // 执行动作
                .andExpect(status().isOk()) // 断言返回状态码为200，如果执行失败则会直接结束
                .andReturn(); // 返回执行完后的结果
    }
}
```
另外，其实测试controller还有一个方法，可以利用chrome浏览器上的插件postman来模拟http请求，感兴趣可以自己去了解。
# 打包
由于我们使用的是Spring Boot，所以可以非常方便地进行打包。