# Conexión a base de datos PostgreSQL e chamada a unha función
## Conexión a base de datos postgre
O primeiro é engadir as dependencias no arquivo pom.xml para obter o driver JBDC de conexión a unha base de datos PostgreSQL.

```xml
    <dependencies>
        <dependency>
            <groupId>postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>9.1-901-1.jdbc4</version>
        </dependency>
    </dependencies>
```
Para conectarnos a base de datos necesitamos (como na maioría de conexións a base de datos) a **URL**, o **nome da base de datos** e as **credenciais**.

```java
//URL e base de datos a cal nos conectamos
String url = new String("192.168.56.102");
String db = new String("test");

//Indicamos as propiedades da conexión
Properties props = new Properties();
props.setProperty("user", "accesodatos");
props.setProperty("password", "abc123.");

//Dirección de conexión a base de datos
String postgres = "jdbc:postgresql://"+url+"/"+db;

//Conectamos a base de datos
try {
    Connection conn = DriverManager.getConnection(postgres,props);
    
    //Cerramos a conexión coa base de datos
    if(conn!=null) conn.close();
} catch (SQLException ex) {
    System.err.println("Error: " + ex.toString());
}
```
## Chamada a unha función de PostgreSQL
É posible utilizar unha función de PostgreSQL para que nos realice unha operación. Neste caso imos crear a función en código dende JAVA, inda que isto non é moi lóxico. O normal e que estivera xa creada ou se creara doutro xeito (polo admin da base de datos por exemplo). Só é para poder realizar o exemplo.

```java
//Creamos a sentencia SQL para crear unha función
//NOTA: nón é moi lóxico crear funcións dende código. Só o fago para despois utilizala
String sqlCreateFucction = new String(
    "CREATE OR REPLACE FUNCTION inc(val integer) RETURNS integer AS $$ "+
    "BEGIN "+
    "RETURN val + 1; "+
    "END;"+
    "$$ LANGUAGE PLPGSQL;");

//Executamos a sentencia SQL anterior
CallableStatement createFunction = conn.prepareCall(sqlCreateFucction);
createFunction.execute();
createFunction.close();
```

A linguaxe que se utiliza para a creación de funcións é PLPSQL. A función do exemplo anterior recibe un enteiro e devolve o valor seguinte.

Agora vemos o código para utilizar dita función.

```java
//Creamos a chamada a función
String sqlCallFunction = new String("{? = call inc( ? ) }");
CallableStatement callFunction = conn.prepareCall(sqlCallFunction);

//O primeiro parámetro indica o tipo de datos que devolve
callFunction.registerOutParameter(1, Types.INTEGER);

//O segundo parámetro indica o valor que lle pasamos a función, neste exemplo 5
callFunction.setInt(2,5);

//Executamos a función
callFunction.execute();

//Obtemos o valor resultante da función
int valorDevolto = callFunction.getInt(1);
callFunction.close();

//Mostramos o valor devolto
System.out.println("Valor devolto da función: " +  valorDevolto);

//Cerramos a conexión coa base de datos
if(conn!=null) conn.close();
```





