# Design Patterns In C#
## Solid Principles
### Interface Segregation Principle

*An interface should be segregated so that nobody that implements an interface has to implement functions which they don't  need*

Consider a modern printer with several functions:
* Printing
* Scanning
* Fax

```csharp
public class Document
{
	// ...
}

public interface IMachine
{
    void Print(Document d);
    void Scan(Document d);
    void Fax(Document d);
}
```

For a modern printer, the implementation will be:

```csharp
public class MultiFunctionPrinter : IMachine
{
    public void Print(Document d)
    {
		// Used
    }
    
	public void Fax(Document d)
    {
    	// Used
    }

    public void Scan(Document d)
    {
        // Used
    }
}
```

But for a basic old Printer, only the printing functionality will be available

```csharp
public class MultiFunctionPrinter : IMachine
{
    public void Print(Document d)
    {
        // Used
    }
    
	public void Fax(Document d)
    {
        // Not Used
    }


    public void Scan(Document d)
    {
        // Not Used
    }
}
```

Solution: Instead of creating an interface with several methods, break those down to serve only what classes need

```csharp
public interface IPrinter
{
    void Print(Document d);
}

public interface IScanner
{
    void Scan(Document d);
}

public interface IFax
{
    void Fax(Document d);
}
```

So that if you want to create a Photocopier without a fax feature, you only implement the behaviour that class uses:

```csharp
public class Photocopier : IPrinter, IScanner
{
    public void Print(Document d)
    {
        // Used
    }

    public void Scan(Document d)
    {
        // Used
    }
}
```