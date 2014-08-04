# Mapping Json

Spin can deserialize json to java objects and serialize java objects to json by integrating Jackson's mapping features into its fluent API.

## Mapping between Representations:

Assume we have a class `Customer` defined as follows:

```java
public class Customer {

  private String name;
  
  public String getName() {
    return name;
  }
  
  public void setName(String name) {
    this.name = name;
  }
}
```

### Mapping Json to Java:

We can map a json object `{"name" : "Kermit"}` to an instance of `Customer` as follows:

```java
import static org.camunda.spin.Spin.JSON;

Customer customer = JSON("{\"customer\": \"Kermit\"}").mapTo(Customer.class);
```

### Mapping Java to Json:

We can map the `customer` back to json as follows:

```java
import static org.camunda.spin.Spin.JSON;

String json = JSON(customer).toString();
```

You can also map Java primitives like boolean or number values to the corresponding json values. For example, `JSON(true)` maps to the json constant of the boolean value `true`. Note however, that String values are not converted but are interpreted as Json input (see [Reading Json][reading-json]). For example, `JSON("a String")` raises an exception because `"a String"` lacks additional escaped quotes and is no valid Json. Nevertheless, a list of String values is properly converted to a json array of String values.

## Mapping to Generic Types:

Assume we have a list of customers that we would declare as `List<Customer>` in Java. For mapping a json array `[{"name" : "Kermit"}, {"name" : "Hugo"}]` to such a list, calling `mapTo(ArrayList.class)` is not sufficient as Jackson cannot tell of which type the array's elements are. This case can be handled by providing `mapTo` with a canonical type string, following Jackson's conventions:

```java
import static org.camunda.spin.Spin.JSON;

String json = "[{\"customer\": \"Kermit\"}, {\"customer\": \"Kermit\"}]"

List<Customer> customers = JSON("{\"customer\": \"Kermit\"}").mapTo("java.util.ArrayList<somepackage.Customer>");
```

## Mapping to Polymorphic Types:

Mapping json to Java objects is particularly tricky as json does not contain type information which is required to deserialize it to Java objects. In the above examples, we have explicitly told Spin to map the json object to a `Customer` object using the `mapTo` method. For nested json objects, Jackson is able to infer the desired deserialization type by inspecting the declared fields of the supplied class. However, this does not work for polymorphic types. Consider the following example, where the `Customer` has a reference to a `Car`. 

```java
public class Customer {

  private String name;
  private Car car;
  
  public String getName() {
    return name;
  }
  
  public void setName(String name) {
    this.name = name;
  }
  
  public Car getCar() {
    return car;
  }
  
  public void setCar(Car car) {
    this.car = car;
  }
}
```

Assuming that `Car` is an interface with various implementations, such as `StationWagon` or `Van`, Jackson cannot tell which implementation to use based solely on the static structure of `Customer`. In these cases, Jackson relies on type information that is part of the json. See the [Jackson documentation][jackson-polymorphy] for the various options Jackson offers to configure type serialization and deserialization. 

If you opt for a default type serialization configuration, you can set it as follows in Spin:

```java
import static org.camunda.spin.Spin.JSON;

String json = 
  "{\"customer\": \"Kermit\", 
    \"car\": 
      {\"brand\": \"Mega Cars\",
      \"@class\": \"somepackage.Van\"}
    }";

Customer customer = 
  JSON(json, 
    jsonTree()
      .mapper()
      .enableDefaultTyping(DefaultTyping.OBJECT_AND_NON_CONCRETE, As.PROPERTY)
      .done())
    .mapTo(Customer.class);
```

The available parameters correspond to Jackson's `DefaultTyping` and `As` enumerations.

When mapping such formatted json to java, the same typing configuration can be applied.

## Date Serialization

Serializing and Deserializing dates typically requires specifying a certain date format (see [Jackson documentation][jackson-date] for built-in date handling). You can set an instance of `DateFormat` by configuring the json data format:

```java
public class DateContainer {

  private Date date;
  
  public Date getDate() {
    return date;
  }
  
  public void setDatae(Date name) {
    this.date = date;
  }
}
```

```java
import static org.camunda.spin.Spin.JSON;

String json = "{\"date\": \"2012-10-10T11:18:42\"}";

DateFormat desiredFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");

DateContainer dateObject = 
  JSON(json, 
    jsonTree()
      .mapper()
      .dateFormat(desiredFormat)
      .done())
    .mapTo(DateContainer.class);
```

[jackson-polymorphy]: http://wiki.fasterxml.com/JacksonPolymorphicDeserialization
[jackson-date]: http://wiki.fasterxml.com/JacksonFAQDateHandling
[reading-json]: reading-json.md