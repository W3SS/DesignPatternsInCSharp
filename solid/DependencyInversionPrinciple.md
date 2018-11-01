# Design Patterns In C#
## Solid Principles
### Dependency Inversion Principle

* **High level parts of the system should not depend on low level parts of the system. Instead, both should depend on abstractions** 
* **Abstractions should not depend on details. Details should depend on abstractions**

Say that we have an *enum Relationship* and a *Class Person* that corresponds to our *High Level Parts*
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

And the class Relationships that corresponds to our *Low Level Parts*

```csharp
public class Relationships
{
	private List<(Person, Relationship, Person)> relations 
            = new List<(Person, Relationship, Person)>();

    public void AddParentAndChild(Person parent, Person child)
    {
        relations.Add((parent, Relationship.Parent, child));
        relations.Add((child, Relationship.Child, parent));
    }

    public List<(Person, Relationship, Person)> Relations => relations;

}
```

If we wanted to search for every relation that contains a **Parent** called **"John"**, we could create an even further *Lower Level Part* that would have a constructor with a **Relationships** object as an argument

```csharp
public class Research
{
	public Research(Relationships relationships)
	{
		var relations = relationships.Relations;
		foreach(var r in relations.where(
			x => x.Item1.Name == "John" &&
				x.Item2 == Relationship.Parent 
			)) 
		{
			Console.WriteLine($"John has a child called {r.Item3.Name}");
		}
	}
}
```

```csharp
static void Main(string[] args)
{
    var parent = new Person {Name = "John"};
    var child1 = new Person {Name = "Chris"};
    var child2 = new Person {Name = "Mary"};

    var relationships = new Relationships();
    relationships.AddParentAndChild(parent, child1);
    relationships.AddParentAndChild(parent, child2);

    new Research(relationships);
}
```

The way it's build works as it is. However we can refactor using the **Dependency Inversion Principle** in a way that it would use interfaces to communicate between *parts* instead of doing it directly.

Instead of using the public method
```csharp
public List<(Person, Relationship, Person)> Relations => relations;
```

We should create an interface 


```csharp
public interface IRelationshipBrowser
{
    IEnumerable<Person> FindAllCHildrenOf(string name);
}
```

Implement the interface in the **Relationships** class and remove the public method **Relations**

```csharp
...
// Remove this method
public List<(Person, Relationship, Person)> Relations => relations;
...
```

And implement the interface method

```csharp

public class Relationships : IRelationshipBrowser
{
    private List<(Person, Relationship, Person)> relations 
        = new List<(Person, Relationship, Person)>();

    public void AddParentAndChild(Person parent, Person child)
    {
        relations.Add((parent, Relationship.Parent, child));
        relations.Add((child, Relationship.Child, parent));
    }

    public IEnumerable<Person> FindAllCHildrenOf(string name)
    {
        foreach (var r in relations.Where(
            x => x.Item1.Name == name &&
            x.Item2 == Relationship.Parent
        ))
        {
            yield return r.Item3;
        }
    }
}
```

By moving the implementation from the **Research** *constructor* to an implementation of the interface in the **Relationships** class, we moved the logic from this lower level class to a higher one 

```csharp
public class Research
{
    public Research(IRelationshipBrowser browser)
    {
        foreach(var p in browser.FindAllCHildrenOf("John"))
        {
            Console.WriteLine($"John has a child called {p.Name}");
        }
    }
    ...
}
```

Producing the same result