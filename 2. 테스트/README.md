## 테스트의 필요성

```
1. 코드가 기대한 대로 동작하는지 검증하기 위해 테스트 코드가 필요.
2. 특히, 스프링과 같은 프레임워크 기반 개발에서는 다양한 설정과 환경에 따라 코드가 예상치 못한 동작을 할 가능성이 높음.
3. 따라서 자동화된 테스트를 도입하여 코드의 신뢰성을 높이고 유지보수를 쉽게 만듬.
```

### 자동화된 테스트의 장점
```
1. 빠른 피드백 제공: 코드 변경 시 즉각적인 검증이 가능.
2. 버그 예방 및 수정 용이: 기존 기능이 깨지지 않았는지 확인 가능.
3. 리팩토링의 안전성 보장: 코드 수정 시 기존 동작을 유지하는지 확인.
4. 문서화 역할: 테스트 코드가 곧 사용 예제 역할을 함.
```

## UserDao 테스트

UserDao의 add() 및 get() 메서드를 검증하는 JUnit 테스트를 작성

```
public class UserDaoTest {
    public static void main(String[] args) throws SQLException {
        UserDao dao = new UserDao();
        
        User user = new User("1", "홍길동", "password123");
        dao.add(user);
        
        System.out.println(user.getId() + " 등록 성공");

        User retrievedUser = dao.get(user.getId());
        if (!user.getName().equals(retrievedUser.getName()) ||
            !user.getPassword().equals(retrievedUser.getPassword())) {
            System.out.println("테스트 실패!");
        } else {
            System.out.println("조회 테스트 성공");
        }
    }
}
```
<br/>

🚨 문제점
```
1. System.out.println()을 사용한 테스트는 수동으로 결과를 확인해야 하므로 자동화가 어렵다.
2. 반복적으로 실행할 수 없으며, 여러 테스트를 일괄 실행하기 어렵다.
3. 예외 발생 시 정확한 원인을 파악하기 어렵다.
```

### JUnit을 이용한 테스트 코드 작성

```
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import java.sql.SQLException;

public class UserDaoTest {

    @Test
    public void addAndGet() throws SQLException {
        UserDao dao = new UserDao();
        User user = new User("1", "홍길동", "password123");
        dao.add(user);
        
        User retrievedUser = dao.get(user.getId());

        assertEquals(user.getName(), retrievedUser.getName());
        assertEquals(user.getPassword(), retrievedUser.getPassword());
    }
}
```
<br/>

개선점
```
1. assertEquals()를 이용해 값이 일치하는지 자동으로 검증.
2. JUnit이 자동으로 실행하므로 수동 확인이 불필요.
3. 예외가 발생하면 JUnit이 자동으로 테스트 실패 원인을 출력.
```

## 테스트의 효율적 관리

### 테스트의 독립성 보장
```
1. 각 테스트는 독립적으로 실행되어야 한다.
2. 하나의 테스트가 다른 테스트에 영향을 주지 않도록 해야 한다.
3. 해결책: 테스트 실행 전후에 데이터 초기화 및 정리 작업 수행.
```

### @BeforeEach와 @AfterEach 활용

JUnit의 @BeforeEach와 @AfterEach 애노테이션을 활용하여 각 테스트 실행 전후에 특정 작업을 수행할 수 있다.

```
import org.junit.jupiter.api.*;

public class UserDaoTest {
    private UserDao dao;

    @BeforeEach
    public void setUp() {
        dao = new UserDao();
    }

    @AfterEach
    public void tearDown() {
        dao.deleteAll();
    }

    @Test
    public void addAndGet() throws SQLException {
        User user = new User("1", "홍길동", "password123");
        dao.add(user);

        User retrievedUser = dao.get(user.getId());

        assertEquals(user.getName(), retrievedUser.getName());
        assertEquals(user.getPassword(), retrievedUser.getPassword());
    }
}
```

## 스프링 테스트 적용

### 테스트를 위한 애플리케이션 컨텍스트 관리
스프링의 테스트 컨텍스트 프레임워크를 활용하면 테스트 시 스프링 컨테이너를 생성하고 필요한 빈을 자동으로 주입할 수 있다.

<br/>

### 스프링 테스트 컨텍스트 프레임워크 적용

스프링은 @SpringBootTest 또는 @ContextConfiguration 애노테이션을 활용하여 테스트 실행 시 애플리케이션 컨텍스트를 로드할 수 있다.

```
@SpringBootTest
public class UserDaoTest {
    @Autowired
    private UserDao userDao;

    @Test
    public void addAndGet() {
        User user = new User("1", "홍길동", "password123");
        userDao.add(user);

        User retrievedUser = userDao.get(user.getId());
        assertEquals(user.getName(), retrievedUser.getName());
        assertEquals(user.getPassword(), retrievedUser.getPassword());
    }
}
```

### 테스트 메소드의 컨텍스트 공유
스프링은 테스트 클래스나 메서드 간에 애플리케이션 컨텍스트를 공유할 수 있도록 지원한다.
@TestInstance(TestInstance.Lifecycle.PER_CLASS)를 사용하면 테스트 실행 간 컨텍스트를 공유하여 실행 속도를 향상시킬 수 있다.

```
@SpringBootTest
@TestInstance(TestInstance.Lifecycle.PER_CLASS)  // 클래스 단위로 컨텍스트 공유
public class UserDaoTest {
    @Autowired
    private UserDao userDao;

    @Test
    public void addAndGet() {
        User user = new User("1", "홍길동", "password123");
        userDao.add(user);
        User retrievedUser = userDao.get(user.getId());
        assertEquals(user.getName(), retrievedUser.getName());
    }
}
```


