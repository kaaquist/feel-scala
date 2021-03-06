# FEEL Scala

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.camunda.feel/feel-engine/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.camunda.feel/feel-engine)

A parser and interpreter for FEEL that is written in Scala (see [What is FEEL?](https://camunda.github.io/feel-scala/develop/what-is-feel)).

The FEEL engine started as a slack time project, grown to a community-driven project and is now officially maintained by [Camunda](https://camunda.org/) :rocket: 

It is integrated in the following projects:
* [Camunda BPM Platform](https://docs.camunda.org/manual/user-guide/dmn-engine/feel/) as part of the DMN engine
* [Zeebe](https://docs.zeebe.io/reference/expressions.html#the-expression-language) as expression language

**Features:** :sparkles:

* full support for unary-tests and expressions (DMN 1.2)
* including built-in functions
* extensible by own functions and custom object mappers

## Usage

The [documentation](https://camunda.github.io/feel-scala/) describes the supported FEEL expressions (e.g. data types, language constructs, builtin-functions, etc.), how to extend/customize the engine and contains some examples.

## Install

The FEEL engine can be used/integrated in two different ways
* as a native Scala application (i.e. calling the engine API)
* as a script engine (i.e. using the Java script engine API)

Add the engine as dependency to your project's `pom.xml`:

```xml
<dependency>
  <groupId>org.camunda.feel</groupId>
  <artifactId>feel-engine</artifactId>
  <version>${VERSION}</version>
</dependency>
```

Or, download the [JAR file](https://github.com/camunda/feel-scala/releases) _(feel-engine-${VERSION}-complete.jar)_ and copy it into your application.

### As Native Scala Application

Create a new instance of the class 'FeelEngine'. Use this instance to parse and evaluate a given expression or unary tests. 

Using Scala:

```scala
object MyProgram {
  
  val engine = new FeelEngine
  
  def feel(expression: String, context: Map[String, Any]) {
    
    val result: Either[Failure, Boolean] = engine.evalSimpleUnaryTests(expression, context)
    // or    
    val result: Either[Failure, Any] = engine.evalExpression(expression, context)
  
    // handle result
    result
        .right.map(value => println(s"result is: $value"))
        .left.map(failure => println(s"failure: $failure"))
  }  
}
```

Or using Java:

```java
 public class MyProgram {

    public static void main(String[] args) {

        final FeelEngine engine = new FeelEngine.Builder()
            .valueMapper(SpiServiceLoader.loadValueMapper())
            .functionProvider(SpiServiceLoader.loadFunctionProvider())
            .build();

        final Map<String, Object> variables = Collections.singletonMap("x", 21);
        final Either<FeelEngine.Failure, Object> result = engine.evalExpression(expression, variables);

        if (result.isRight()) {
            final Object value = result.right().get();
            System.out.println("result is " + value);
        } else {
            final FeelEngine.Failure failure = result.left().get();
            throw new RuntimeException(failure.message());
        }
    }
}
```

Enable FEEL External-Functions in the configuration: 

```
new FeelEngine(configuration = Configuration(externalFunctionsEnabled = true))
// or
new FeelEngine.Builder().enableExternalFunctions(true).build()
```

:warning: Enabling external functions is not recommended because external functions make it possible to call arbitrary Java methods. This poses a potential security risk. Enabling this feature can be safe if you have full control of the expressions that will be evaluated (i.e. none of them are entered by a user or generated by an external system). In all other cases this feature is considered unsafe. It is recommended to use the [Function Provider SPI](https://camunda.github.io/feel-scala/develop/function-provider-spi) instead.

### As Script Engine

Call the engine via Java's script engine API ([JSR 223](https://www.jcp.org/en/jsr/detail?id=223)).

```scala
object MyProgram {

  val scriptEngineManager = new ScriptEngineManager
 
  def feel(script: String, context: ScriptContext) {
  
    val scriptEngine: FeelScriptEngine = scriptEngineManager.getEngineByName("feel")
    
    val result: Object = scriptEngine.eval(script, context)
    // ...
  }

}
```

The engine is registered under the names:

* `feel`
* `http://www.omg.org/spec/FEEL/20140401` (qualified name)
* `feel-scala`

You can also evaluate unary tests instead of an expression by using one of the names:

* `feel-unary-tests`
* `feel-scala-unary-tests`

## Contribution

See the [Contribution Guide](./CONTRIBUTING.md).

The following resources can help to understand the general concepts behind the implementation: 
* [Scala's Combinator Parsing](https://www.artima.com/pins1ed/combinator-parsing.html)
* [Build your own Programming Language with Scala](https://www.lihaoyi.com/post/BuildyourownProgrammingLanguagewithScala.html)

## License

[Apache License, Version 2.0](./LICENSE)
