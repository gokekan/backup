MySQL 접속유지를 위한 스프링프레임워크 설정
===

# 서론
MySQL은 기본적으로 wait_timeout이 28800, 기본적으로 8시간을 기다린 후, 접속을 끊습니다.

일반적으로 저 시간 내에 충분히 요청이 오기 때문에 접속이 끊기지 않고 유지가 됩니다. 하지만 대기 시간내에 접속이 없으면, 접속이 끊어지는데 반대로 Connection Pool에는 접속을 가지고 있는 상태로 여기기 때문에, 호출이 오면 오류가 발생합니다.

이런 상황을 미연에 방지하고자 이 글을 작성하였습니다.

# MySQL wait_timeout 보기
    mysql> show variables like '%timeout'
	+----------------------------+-------+
	| Variable_name              | Value |
	+----------------------------+-------+
	| connect_timeout            | 10    |
	| delayed_insert_timeout     | 300   |
	| innodb_lock_wait_timeout   | 50    |
	| innodb_rollback_on_timeout | OFF   |
	| interactive_timeout        | 28800 |
	| net_read_timeout           | 30    |
	| net_write_timeout          | 60    |
	| slave_net_timeout          | 3600  |
	| table_lock_wait_timeout    | 50    |
	| wait_timeout               | 28800 |
	+----------------------------+-------+
	10 rows in set (0.01 sec)


# 스프링프레임워크 데이터소스 설정

	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<!-- 접속정보 -->
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/dbname" />
		<property name="username" value="user" />
		<property name="password" value="password" />

		<!-- 접속관련 설정 -->
		<property name="initialSize" value="5"/>	
        <property name="maxActive" value="20"/>	
        <property name="minIdle" value="5"/>
        <property name="maxWait" value="3000"/>	
        <property name="poolPreparedStatements" value="true"/>
        <property name="maxOpenPreparedStatements" value="50"/>

		<!-- 특정 시간(2시간)마다 validationQuery를 실행 -->
		<property name="validationQuery" value="select 1"/>
		<property name="testWhileIdle" value="true"/>
        <property name="timeBetweenEvictionRunsMillis" value="7200000"/> 
	</bean> 

## DB Connection Pool 설정 

### maxActive
Connection Pool이 제공할 최대 접속 수.

### maxIdle
사용되지 않고 Connection Pool에 저장될 수 있는 최대 접속 수. 음수 일 경우에는 제한이 없다.

### minIdle
사용되지 않고 Connection Pool에 저장될 수 있는 최소 접속 수.

### maxWait
접속 사용이 많아져서 Connection Pool이 비었을 때, 사용할 수 있는 접속을 받기까지 기다릴 수 있는 최대 시간.

### testOnBorrow
true 일 경우, Connection Pool 에서 접속을 가져올 때, 유효한 접속인지를 검사한다.

### testOnReturn
true 일 경우, Connection Pool 에서 접속을 반환할 때, 유효한 접속인지를 검사한다.

### ★ testWhileIdle ()
true 일 경우, 접속에 대한 유효성 검사를 Connection Pool에 Idle 상태가 존재할 때 실시한다. 반드시, validationQuery가 설정되어 있어야 된다.

### ★ validationQuery
접속 유효성 검사 시, 사용할 쿼리문을 설정한다. DB 리소스를 최대한 적게 사용하는 쿼리를 사용하는 것이 좋다.

### ★ timeBetweenEvctionRunsMillis
사용하지 않는 접속을 제거하는 쓰레드의 실행 주기를 지정한다. 양수가 아닐 경우는 실행되지 않는다. 단위는 1/1000초 이다.

### poolPreparedStatement
Prepared Statement 사용여부. 자주 사용되는 쿼리를 미리 Common Place(Pool) 에 저장해 놓는 것 이다. true일 경우, 반드시 maxOpenPreparedStates를 설정해야한다.

### maxOpenPreparedStatements
Prepared Statement Pool에 저장할 최대 Statement 수. 기본값은 "unlimited" 이다. 이 값이 적절하지 않으면, 런타임 시 Out of Memory 등의 에러가 발생할 수 있다.
 