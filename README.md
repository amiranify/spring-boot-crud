# spring-boot-crud
## Задание
Необходимо написать приложение, которое способно хранить список
пользователей.
Данное приложение будет иметь весь функционал CRUD-приложения. Все
операции должны производиться в браузере, а именно:
1. иметь возможность просматривать список пользователей
2. добавлять пользователей в список
3. обновлять существующих пользователей (например, изменять имя)
## Какие dependency нам понадобятся?
**Spring-boot-starter-web** - стартер, который
содержит основные зависимости для
реализации веб-приложения.

**Spring-boot-starter-thymeleaf** - стартер,
который содержит основные зависимости
для корректной работы шаблонизатора Thymeleaf

**Spring-boot-starter-data-jpa** -  стартер, необходим для работы с БД.

**h2** - необходимая зависимость для
поднятия h2
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
 ```
## Конфигурация приложения

Так как мы используем БД, потребуется передать необходимые настройки
для корректного взаимодействия приложения и БД. Данные настройки
должны описать в файле application.properties:
| **код** | **описание** |
| --- | --- |
| spring.datasource.url =jdbc:h2:mem:users | указываем необходимое jdbc - подключение |
| spring.datasource.driverClassName =org.h2.Driver | указываем jdbc-драйвер для корректного взаимодействия |
| spring.jpa.database-platform =org.hibernate.dialect.H2Dialect | указываем диалект взаимодействия |
## Описание сущности User
Наша сущность будет хранить 3 поля:
- id - первичный ключ
- firstName - имя
- lastName - фамилию
 
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "first_name")
    private String firstName;
    @Column(name = "last_name")
    private String lastName;
    // Геттеры и сеттеры, конструкторы опущены для наглядности
}
```
## Создание JPA - репозитория.
Данная сущность необходима для взаимодействия с БД:
```java
public interface UserRepository extends JpaRepository<User , Long> {
}
```
- **User** - сущность, с которой будет взаимодействовать данный JPA -
репозиторий.
- **Long** - тип первичного ключа.

## UserService
Данный сервис является необязательным, так как в нашем приложении
отсутствует логика. Однако при расширении функционала приложения вся
логика будет в нем находиться.
**Реализация:**
```java
@Service
public class UserService {
 private final UserRepository userRepository ;
 @Autowired
 public UserService (UserRepository userRepository) {
 this.userRepository = userRepository ;
 }
 public User findById(Long id){
 return userRepository .getOne(id) ;
 }
 public List<User> findAll(){
 return userRepository .findAll();
 }
 public User saveUser(User user){
 return userRepository .save(user) ;
 }
 public void deleteById(Long id){
 userRepository .deleteById(id) ;
 }
}
```
## Controller
С этим компонентом мы уже знакомы. Данный компонент необходим для
взаимодействия пользователя с приложением по http. Контроллеру
необходим UserService для взаимодействия с БД.
### Создание контроллера 
Запросы для реализации CRUD - функционала. Первая часть
```java
@Controller
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/users")
    public String findAll(Model model) {
        List<User> users = userService.findAll();
        model.addAttribute("users", users);
        return "user-list";
    }

    @GetMapping("/user-create")
    public String createUserForm(User user) {
        return "user-create";
    }

    @PostMapping("/user-create")
    public String createUser(User user) {
        userService.saveUser(user);
        return "redirect:/users";
    }

    @GetMapping("user-delete/{id}")
    public String deleteUser(@PathVariable("id") Long id) {
        userService.deleteById(id);
        return "redirect:/users";
    }

    @GetMapping("/user-update/{id}")
    public String updateUserForm(@PathVariable("id") Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", user);
        return "user-update";
    }

    @PostMapping("/user-update")
    public String updateUser(User user) {
        userService.saveUser(user);
        return "redirect:/users";
    }
}
```
## Создание View
1. View для просмотра списка пользователей *user-list*
2. View для создания пользователя *user-create*
3. View для обновления пользователя *user-update*

**user-list**
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Users</title>
</head>
<body>
<div th:switch="${users}">
    <h2 th:case="null">No users found!</h2>
    <div th:case="*">
        <h2>Users</h2>
        <table>
            <thead>
            <tr>
                <th>Id</th>
                <th>First name</th>
                <th>Last name</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="user : ${users}">
                <td th:text="${user.id}"></td>
                <td th:text="${user.firstName}"></td>
                <td th:text="${user.lastName}"></td>
                <td><a th:href="@{user-update/{id}(id=${user.id})}">Edit</a></td>
                <td><a th:href="@{user-delete/{id}(id=${user.id})}">Delete</a></td>
            </tr>
            </tbody>
        </table>
    </div>
    <p><a href="/user-create">Create user</a></p>
</div>
</body>
</html>
```
**user-create**
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Create user</title>
</head>
<body>
<form action="#" th:action="@{/user-create}" th:object="${user}" method="post">
    <label for="firstName">First name</label>
    <input type="text" th:field="*{firstName}" id="firstName" placeholder="First Name">
    <label for="lastName">Last name</label>
    <input type="text" th:field="*{lastName}" id="lastName" placeholder="Last Name">
    <input type="submit" value="Create User">
</form>
</body>
</html>
```
**user-update**
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<meta charset="UTF-8">
<title>Create user</title>
</head>
<body>
<form action="#" th:action="@{/user-update}" th:object="${user}" method="post">
    <label for="id">ID</label>
    <input readonly type="number" th:field="*{id}" id="id" placeholder="ID">
    <br/>
    <label for="firstName">First name</label>
    <input type="text" th:field="*{firstName}" id="firstName" placeholder="First Name">
    <br/>
    <label for="lastName">Last name</label>
    <input type="text" th:field="*{lastName}" id="lastName" placeholder="Last Name">
    <br/>
    <input type="submit" value="Update User">
</form>
</body>
</html>
```
