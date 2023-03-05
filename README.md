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
