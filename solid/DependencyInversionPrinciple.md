# Design Patterns In C#
## Solid Principles
### Dependency Inversion Principle

* **High level parts of the system should not depend on low level parts of the system. Instead, both should depend on abstractions** 
* **Abstractions should not depend on details. Details should depend on abstractions**

Say that we have an *enum Relationship* and a *Class Person* that corresponds to our High Level
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

And the class Relationships that corresponds to our Low Level

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

So if we wanted to get a **Parent** named **John** we could create a Class **Research** with a constructor like this: 

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

If you declare 3 **Person** objects it will work just fine the way it is

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

However, the problem is that the relationship class cannnot be changed without rewritting some of the code. By using the **Dependency Inversion Principle**, we need to add a intermediary interface in the Relationship class to access high level data. On this particular example is the 

```csharp
public List<(Person, Relationship, Person)> Relations => relations;
```

in the **Relationships** class.
Creating an interface

```csharp
public interface IRelationshipBrowser
{
    IEnumerable<Person> FindAllCHildrenOf(string name);
}
```

We can implement the interface in the **Relationships** class and remove the public method **Relations**

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