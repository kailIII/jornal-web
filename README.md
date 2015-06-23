# jornal-web
Sistema de Jornal Web utilizando Spring Framework, Hibernate e Bootstrap


###Configuração do Ambiente
####Deve-se acrescentar a linha em destaque no arquivo server.xml:

```xml
<Resource auth="Container" type="javax.sql.DataSource" 
          maxActive="10" maxIdle="3" maxWait="10000"
          driverClassName="org.postgresql.Driver" name="jornal"
          username="postgres" password="postgres"
          url="jdbc:postgresql://localhost/jornal" />
```

####Deve-se acrescentar a linha em destaque no arquivo context.xml:

```xml
<ResourceLink global="jornal" name="jornal" type="javax.sql.DataSource"/>
```

###Resultado Final

Os recursos serão disponibilizados pelo Tomcat com o seguinte nome JNDI:

```xml
java:comp/env/jdbc/nome_do_datasource
```

###Alterações na aplicação que utiliza Spring Framework para usar as configurações de JNDI:

####1. No applicationContext.xml é preciso dizer que a aplicação vai usar o dataSource disponibilizado pelo Tomcat (java:comp/env/jdbc/nome_do_datasource):

```xml
<bean id="jornal" class="org.springframework.jndi.JndiObjectFactoryBean">
		<property name="jndiName" value="java:comp/env/jornal" />
	</bean>
	<bean id="emf"
		class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="persistenceUnitName" value="dev" />
		<property name="dataSource" ref="jornal" />
	</bean>
```
Perceba que o entityManagerFactory vai referenciar esse mesmo dataSource.

#### 2. No spring-security.xml também é preciso dizer que ele deve usar o dataSource definido no applicationContext.xml:

```xml
<authentication-manager>
		<authentication-provider>
			<password-encoder hash="sha-256"/>
			<jdbc-user-service data-source-ref="jornal" ... />
```


####3. No persistence.xml devemos remover as configurações do banco de dados que estamos usando:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.0"
      xmlns="http://java.sun.com/xml/ns/persistence"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/persistence 
               http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">

  <persistence-unit name="dev" transaction-type="RESOURCE_LOCAL">
	<provider>org.hibernate.ejb.HibernatePersistence</provider>

    <properties>
	  <property name="hibernate.dialect" value="org.hibernate.dialect.PostgreSQLDialect"/>
      <property name="hibernate.hbm2ddl.auto" value="update" />
      <property name="hibernate.show_sql" value="true" />
      <property name="hibernate.format_sql" value="true" />
    </properties>
  </persistence-unit>
</persistence>
```
