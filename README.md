# Создание простого RESTful приложения для дальнейшей реализации  JWT аутентификации на его базе

   Недавно, в процессе обучения Java, передо мной стала задача реализовать аутентификацию своего демо приложения через JSON Web Token (JWT). В сети довольно много вариантов решения данной задачи, но доступных и внятных русскоязычных ресурсов
довольно мало и информация на них не всегда соответствует ожидаемому. 
	
По этому я решил опубликовать эту статью основываясь на англоязычных источниках, которые мне любезно выдал Google. Быть может, она кому то пригодиться и действительно в чем то поможет. Так же приветствуються комментарии и замечания, ведь я тоже учусь и они точно не будут лишними.
	
Но, что бы начать реализацию JWT, нам понадобиться какое-то не очень замысловатое приложение. Оно будет имень два метода: создание учетной записи, доступ к которому будут иметь все пользователи, и метод возвращающий приветственную строку, получить доступ к которому смогут только пользователи прошедщее аутентификацию.
	
Назовем наше RESTful приложение testJWT и построим его на базе Spring Boot, с использованием Lombok, PostgreSQL, Liquibase и Hibernate, сделаем вид, что наше приложение - это что то действительно серьезное и создадим таблицу базы данных в ручную. Зависимости Spring Security и JWT прикрутим нашему приложению уже в процессе реализации нашей аутентификации. Приложение выйдет очень простое и не будет нести в себе какого тайного смысла и дополнительных проверок. Так, ввиду 
чисто тестового характера приложения и отсутствия критической необходимости, мы не будем проверять уникальность имен пользователей при их регистрации перед внесением в базу данных.
	
Особо не терпеливые могут пропустить данную статью и сразу перейти к ее второй части (непосредственно реализации JWT аутентификции) клонировав готовое базовое приложе из моего [github](https://github.com/SashaUsov/testJWT.git) репозитория. Так же, если вас интересует реализация аутентификации через JWT, осмелюсь предположить, что создание подобного простого каркасса приложения вам не в новинку. Тем не менее, рассмотрим некоторые аспекты его создания более детально.

Первым делом, зайдем на [SPRING INITIALIZR](https://start.spring.io) и сгенерируем наш проект.

![alt text](https://github.com/SashaUsov/testJWT/issues/1#issue-386542415)

Заполнив форму необходимыми данными и подключив нужные зависимости, импортирум проект в свою любимую IDE. Я буду использовать [IntelliJ IDEA](https://www.jetbrains.com/idea/).

![alt text](https://github.com/SashaUsov/testJWT/issues/2#issue-386542538)

Далее, следую не хитрым подсказкам при настройке импорта, завершаем начатое и даем IDE подгрузить все зависимости.

![alt text](https://github.com/SashaUsov/testJWT/issues/3#issue-386542574)

Не забываем включать автоимпорт, дабы среда сама справлялась с этой задачей вовремя.

![alt text](https://github.com/SashaUsov/testJWT/issues/4#issue-386542592)

Когда импорт проекта завершиться, мы найдем в нем один единственный класс DemoApplication, который пожет стартануть нашему приложению. Чуть позже мы добавим еще логики, а сейчас давайте перейдем в файл build.gradle и попробуем разобрать что же там и для чего.

```java
buildscript {
	ext {
		springBootVersion = '2.1.0.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'testJWT'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-data-jpa')
	implementation('org.springframework.boot:spring-boot-starter-web')
	runtimeOnly('org.springframework.boot:spring-boot-devtools')
	compile('org.liquibase:liquibase-core')
	compile('org.postgresql:postgresql')
	compile('org.projectlombok:lombok')

	testImplementation('org.springframework.boot:spring-boot-starter-test')
}
```
`compile(‘org.springframework.boot:spring-boot-starter-data-jpa')` - помежет нам при работе с базой данных, содержит в себе так же и Hibernate.

`implementation(‘org.springframework.boot:spring-boot-starter-web')` - даст нам использовать все возможност Tomcat и Spring MVC.

`compile(‘org.postgresql:postgresql')` - даст возможность работать с PostgreSQL.

`compile(‘org.liquibase:liquibase-core')` - с помощью этой зависимости мы упростим себе создание таблиц и написание миграй к базе данных.

`compile(‘org.projectlombok:lombok')` -  упростит нам весь процесс создания аксессоров к приватным полям классов по средству аннотаций @Getter и @Setter. Для этого, предворительно загрузим lombok plugin в настройках приложения (Preferences > Plugins).

После успешной установки плагина включим возможность пользоваться  данными аннотациями (Preferences > Build, Execution, Deployment > Annotation Processors) поставив флажок на против Enable annotation processing.

![alt text](https://github.com/SashaUsov/testJWT/issues/6#issue-386542627)

Приступим к созданию логики приложения и наполним его чем то более менее похожим на функциональность. Первым делом создадим сущность пользователя, которую собственно говоря мы и будем заносить в базу данных. Расположение данного класса будет следующим: package testJWT.demo.domain;

Само тело класса будет иметь вид:

```java
package testJWT.demo.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;

@Getter
@Setter
@Entity
@Table(name = "user_table")
public class ApplicationUser {

    @Column(name = "user_name")
    String userName;

    @Column(name = "password")
    String password;

    @Column(name = "id", updatable = false, nullable = false)
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "usr_sequence")
    Long id;
}
```
Создадим package-info.java файл package testJWT.demo.domain; который будет содержать sequence с поведением при генерации id пользователя.

```java
@GenericGenerator(
        name = "usr_sequence",
        strategy = "org.hibernate.id.enhanced.SequenceStyleGenerator",
        parameters = {
                @Parameter(name = "usr_sequence", value = "sequence"),
                @Parameter(name = "initial_value", value = "1"),
                @Parameter(name = "increment_size", value = "1"),
        }
)

package testJWT.demo.domain;

import org.hibernate.annotations.GenericGenerator;
import org.hibernate.annotations.Parameter;
```

Следующим шагом создадим репозиторий в расположении: 
package testJWT.demo.repo;

```java
package testJWT.demo.repo;

import org.springframework.data.jpa.repository.JpaRepository;
import testJWT.demo.domain.ApplicationUser;

public interface UserRepo extends JpaRepository<ApplicationUser, Long> {

}
```

Данный интерфейс даст нам возможность осуществлять разные действия (такие как поиск, удаление, сохранение и т.д.) в таблице базы данных.

Далее предлогаю создать модель, при помощи которой мы будем получать JSON контент от пользоватей с данными необходимыми для регистрации. Его располоим в package testJWT.demo.domain.dto;

```java
package testJWT.demo.domain.dto;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class CreateUserModel {

    private String userName;

    private String password;
}
```

Приступим к создаю сервиса, который будет содержать всю логику прилржения, в нашем случае довольно примитивную. Расположим сервис класс в следующем пакете package testJWT.demo.service;

```java
package testJWT.demo.service;

import org.springframework.stereotype.Service;
import testJWT.demo.domain.ApplicationUser;
import testJWT.demo.domain.dto.CreateUserModel;
import testJWT.demo.repo.UserRepo;

@Service
public class MainService {

    private final UserRepo userRepo;

    public MainService(UserRepo userRepo) {
        this.userRepo = userRepo;
    }

    public String getGreeting() {

        return "You could and created JWT authentication!";
    }

    public ApplicationUser create(CreateUserModel userModel) {

        ApplicationUser applicationUser = new ApplicationUser();

        applicationUser.setUserName(userModel.getUserName());
        applicationUser.setPassword(userModel.getPassword());

        return userRepo.save(applicationUser);
    }
}

```

Аннотация @Service даст понять Spring что данный класс являеться сервисом и для него необходимо создать бин, а иначе ничего у нас не выйдет и не запустится.

Следующим шагом будет создание контроллера, который собственно и будет принимать запросы от пользователей и передавать их на исполнение сервису. Его мы разместим в пакете package testJWT.demo.controller;
	
Аннотация @RestController скажет Spring что этот класс контроллер и что именно он будет принимать и отдавать контент пользователю. Аннотация @RequestMapping(“main") указывает на коком адресе контроллер будет осуществлять свою работу.
	
Посмотрим содержимое данного класса


```java
package testJWT.demo.controller;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import testJWT.demo.domain.ApplicationUser;
import testJWT.demo.domain.dto.CreateUserModel;
import testJWT.demo.service.MainService;

@RestController
@RequestMapping("main")
class MainController {

    private final MainService mainService;

    public MainController(MainService mainService) {
        this.mainService = mainService;
    }

    @GetMapping("greeting")
    public String getGreeting() {

        return mainService.getGreeting();
    }

    @PostMapping("registration")
    @ResponseStatus(HttpStatus.CREATED)
    public ApplicationUser create(@RequestBody CreateUserModel userModel) {

        return mainService.create(userModel);
    }
}
```

На этом вся логика обработки событий окончена и остается только настроить проперти и прописать миграции базы данных.
	
Начнем с application.properties

```properties
spring.datasource.url=jdbc:postgresql://localhost/testjwt
spring.datasource.username=sasha_vosu
spring.datasource.password=1151855
spring.jpa.generate-ddl=false
spring.jpa.hibernate.ddl-auto=none

spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.show-sql=false

spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=false

spring.liquibase.change-log=classpath:/db/changelog-master.yaml
```

Строки с 1 по 5 содержат настройки подключения и авторизации базы данных (напомню, что я использую PostgreSQL и то, что вам будет необходимо создать свою собственную базу данных и обозвать ее как вам будет угодно).

Строка 7 указывает что нужно использовать именно PostgreSQL диалект.

Строка 12 указывает путь, в котором будет храниться файл включающий все наши миграции. По этому, идем в проект и создаем папку с названием db в пакете resources. Размещаем в этой папке сам changelog-master.yaml файл со следующим содержанием

```yaml
databaseChangeLog:

    - include:
        file: db/20181126T1520_create_user_table.yaml
    - include:
        file: db/20181126T1524_usr_sequence.yaml
```

Как мы видим, этот файл включает в себя еще два файла, по этому смело создаем и их в этом же db пакете.

20181126T1520_create_user_table.yaml будет иметь вид
	
```yaml
databaseChangeLog:

  - changeSet:
      id: 1
      author: sasha_vosu
      changes:

      - createTable:
          tableName: user_table
          columns:
          - column:
              name: user_name
              type: varchar(10)
          - column:
              name: id
              type: bigint
              autoIncrement: true
              constraints:
                primaryKey: true
                nullable: false
          - column:
              name: password
              type: varchar
```

Не трудно понять, что данным файлом мы даем команду на создание таблицы для нашей сущности (ApplicationUser), а имя таблицы, колонок и их содержание полностью совпадают с указанными в ранее созданном классе.

Следующим шагом будет создание 20181126T1524_usr_sequence.yaml файла содержащего sequence с поведением при генерации id пользователя (если помните мы создали его в этом месте package testJWT.demo.domain; и назвали 
package-info.java)

```yaml
databaseChangeLog:

    - changeSet:
            id: 1
            author: sasha_vosu
            changes:

            - createSequence:
                incrementBy: 1
                sequenceName: usr_sequence
                startValue: 1
```

На этом, можно считать что мы закончили наше примитивное приложение и уже готовы переходить ко второй статье, но давайте в начале запустим то что вышло и протестируем (скажем к примеру с помощью Postman).

Отправим GET Запрос на следующий URL: <http://localhost:8080/main/greeting>

Если все работает как задумывалось, в ответ мы получим строку

	"You could and created JWT authentication!"
	
Но не стоит обольщаться, это далеко не аутентификация, не верьте этой коварной строке!

Отправив POST запрос на URL <http://localhost:8080/main/registration> с JSON следующего вида 

```json
{ 
   "userName": "admin", 
   "password": “password"
} 
```
  
  мы создадим и внесем сущность пользователя в таблицу.
  
  Из полученного ответа видно, что мы создали пользователя с именем admin, id равным 1 и паролю соответствующему слову password. В принципе, как и хотели.
  
Но почему пароль виден всем? Почему мы смогли получить приветственную строку еще даже не зарегестрировав и не авторизировав пользователя? Это же ужасно не безопасно!

Давайте вооружимся JWT и Spring Security и исправим этот досадный промах! Но только вот уже в следующей статье :)
