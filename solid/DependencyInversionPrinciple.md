### Dependency Inversion Principle

** High level parts of the system should not depend on low level parts of the system directly. Instead it should depend on some kind of abstraction ** 

Say that we have an *enum Relationship* and a *Class Person*
```csharp
public enum Relationship
{
    Parent,
    Child,
    Sibling
}

public class Person
{
    public string Name;
}
```
