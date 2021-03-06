# JED - The Java Deserializer Abstraction Layer

JED is a pluggable Java API that abstracts away the underlying implementation libraries for various data interchange
formats, such as JSON, Avro, etc. This enables libraries that use JED to not have to expose a hard dependency on any 
particular implementation. At present the focus of this library is on deserialization only, with support for two
JSON libraries (Jackson and Gson) and one Avro library (Apache Avro).

## Usage

This library is intended for use by other class libraries that desire to expose deserialization to end users in a 
way that does not infer a hard dependency on a particular implementation library. Without using this 
library, developers may need to expose API such as this:

```java
ClientLibrary client = new ClientLibrary();
client.registerTypeAdapter(Foo.class, new JsonDeserializer<Foo> {
    @Override public Foo deserialize(JsonElement jElement, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        JsonObject jObject = jElement.getAsJsonObject();
        int intValue = jObject.get("valueInt").getAsInt();
        String stringValue = jObject.get("valueString").getAsString();
        return new Foo(intValue, stringValue);
    }
});
```

Unfortunately, this means that our fictional `ClientLibrary` class now finds itself offering API that is tightly coupled
with the Gson `JsonDeserializer` class (as well as other Gson types `JsonElement`, `JsonDeserializationContext`,
`JsonParseException`, and `JsonObject`). This means that we, as the owner of `ClientLibrary`, are now forever bonded to 
Gson, regardless of its ongoing support, popularity, feature set, and security issues. The same argument can be made if 
we instead choose to expose Jackson through our public API. The goal of this library is to offer an abstraction over any
particular library, such that they may be used interchangeably.

Because JED is pluggable, it means that the user can opt-in to using either Gson or Jackson (for JSON), and Apache Avro
(for Avro), or any other compatible plugin. This means that if the user has annotated their model classes with Gson- or 
Jackson-specific annotations, that they can use the appropriate plugin to ensure that these annotations are fully consumed 
as part of the parsing process, to get the desired results.

### JSON

Instead, the API offered to the user should use the `Deserializer` class:

```java
ClientLibrary client = new ClientLibrary();
client.registerTypeAdapter(new Deserializer<Foo>(Foo.class) {
    @Override public Foo deserialize(Node node) {
        int intValue = node.get("valueInt").asInt();
        String stringValue = node.get("valueString").asString();
        return new Foo(intValue, stringValue);
    }
});
```

Internally, the `ClientLibrary` class would perform the following work:

```java
public class ClientLibrary {
    private JsonApi jsonApi;
    
    public ClientLibrary() {
        this.jsonApi = JsonWrapper.newInstance();
    }
    
    public void registerTypeAdapter(Deserializer<?> deserializer) {
        this.jsonApi.registerCustomDeserializer(deserializer);
    }
}
```

In many cases registering a custom Deserializer will be overkill. In many cases it is possible to simply use the `JavaApi`
APIs directly, e.g:

```java
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
Car car = jsonApi.readString(json, Car.class);
```

More examples can be seen in the [jed-json-test](https://github.com/JonathanGiles/jsonwrapper/tree/master/jed-json-test) module.

### Avro

To be continued...