# 초난감 DAO (Data Access Object)

## User 클래스

데이터베이스에서 사용자 정보를 관리하기 위한 User 클래스

```
public class User {
    private String id;
    private String name;
    private String password;

    public User(String id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public String getPassword() { return password; }
} 
```
<br/>

## UserDao 클래스

데이터베이스에 접근하여 사용자 데이터를 추가하거나 조회하는 DAO 클래스

```
public class UserDao {
    public void add(User user) throws SQLException {
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/spring", "root", "password");

        PreparedStatement ps = conn.prepareStatement("INSERT INTO users(id, name, password) VALUES (?, ?, ?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();
        ps.close();
        conn.close();
    }

    public User get(String id) throws SQLException {
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/spring", "root", "password");

        PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
        ps.setString(1, id);
        ResultSet rs = ps.executeQuery();

        rs.next();
        User user = new User(rs.getString("id"), rs.getString("name"), rs.getString("password"));

        rs.close();
        ps.close();
        conn.close();
        return user;
    }
}
```
<br/>

### 🚨 문제점 <br/>
•	DB 연결 코드가 하드코딩되어 있음 → 데이터베이스 설정을 변경하려면 코드 수정이 필요 <br/>
•	관심사의 분리가 되지 않음 → UserDao 클래스는 데이터 처리 로직과 DB 연결 로직을 함께 가지고 있음

<br/>

## DAO의 분리 (관심사 분리)

UserDao가 담당하는 관심사
1. DB 연결
2. SQL 실행 및 결과 처리

이 두 가지가 섞여 있기 때문에 DB 연결 부분을 별도의 메서드로 추출하는 것이 필요

중복되는 DB 연결 코드를 메서드로 분리

```
public class UserDao {
    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection("jdbc:mysql://localhost/spring", "root", "password");
    }

    public void add(User user) throws SQLException {
        Connection conn = getConnection(); // 메서드를 통해 DB 연결
        PreparedStatement ps = conn.prepareStatement("INSERT INTO users(id, name, password) VALUES (?, ?, ?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());
        ps.executeUpdate();
        ps.close();
        conn.close();
    }
}
```

중복 코드는 줄였지만 DB 변경 시 getConnection()을 직접 수정해야 하므로 여전히 유연하지 않음

<br/>

## DB 커넥션 만들기의 독립

DB 연결 방식을 변경하기 위해 상속 도입

```
public class MySqlUserDao extends UserDao {
    @Override
    protected Connection getConnection() throws SQLException {
        return DriverManager.getConnection("jdbc:mysql://localhost/spring", "root", "password");
    }
}
```
상속은 결합도를 높이는 문제가 발생
<br/>

## DAO 확장

### 클래스 분리
DB 연결을 담당하는 별도 클래스를 정의하여 UserDao의 책임을 줄이기

```
public interface ConnectionMaker {
    Connection makeConnection() throws SQLException;
}
```

이를 구현한 클래스를 만들면 DB 연결 전략을 쉽게 바꿀 수 있음

```
public class SimpleConnectionMaker implements ConnectionMaker {
    public Connection makeConnection() throws SQLException {
        return DriverManager.getConnection("jdbc:mysql://localhost/spring", "root", "password");
    }
}
```

UserDao는 ConnectionMaker 인터페이스를 사용하여 유연성을 확보

```
public class UserDao {
    private final ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }

    public void add(User user) throws SQLException {
        Connection conn = connectionMaker.makeConnection();
        PreparedStatement ps = conn.prepareStatement("INSERT INTO users(id, name, password) VALUES (?, ?, ?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());
        ps.executeUpdate();
        ps.close();
        conn.close();
    }
}
```
<br/>

## 객체지향 설계 원칙 (SOLID)

1. 단일 책임 원칙 (SRP) <br/>
	•	UserDao는 데이터베이스와의 연결과 SQL 실행을 모두 책임지는 단일 책임 원칙 위반 상태였다. <br/>
	•	해결책: DB 연결을 ConnectionMaker 인터페이스로 분리.

2. 개방-폐쇄 원칙 (OCP) <br/>
	•	UserDao의 변경 없이 다양한 DB 연결 방식(MySQL, Oracle 등)을 적용할 수 있도록 구성. <br/>
	•	해결책: ConnectionMaker 인터페이스를 통해 확장 가능하도록 설계.

3. 리스코프 치환 원칙 (LSP) <br/>
	•	ConnectionMaker를 구현한 여러 클래스가 동일한 방식으로 교체 가능해야 한다.

4. 인터페이스 분리 원칙 (ISP) <br/>
	•	UserDao가 불필요한 인터페이스를 구현하지 않도록 필요한 기능만 제공하는 인터페이스를 활용.

5. 의존관계 역전 원칙 (DIP) <br/>
	•	UserDao가 ConnectionMaker 인터페이스에 의존하여, 구체적인 구현체가 아닌 추상화된 인터페이스를 활용.

<br/>

## 제어의 역전(IoC)
제어의 역전은 객체의 생성과 관리를 개발자가 아닌 외부에서 담당하는 방식.<br/>
IoC를 통해 객체 간의 결합도를 낮추고, 유연한 코드를 작성할 수 있다.

<br/>

## 스프링의 IoC
스프링은 IoC를 통해 객체의 생성과 관리를 쉽게 할 수 있도록 지원한다.<br/>
 스프링의 IoC 컨테이너는 빈(Bean)이라고 불리는 객체를 생성하고 관리.

```
 ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
UserDao userDao = context.getBean("userDao", UserDao.class);
```