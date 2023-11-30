C# Coding Style
===============

Based on :
https://google.github.io/styleguide/csharp-style.html
https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md

Rule summary:
Code

   - Names of classes, methods, enumerations, public fields, public properties, namespaces: PascalCase.
   - Names of local variables, parameters: camelCase.
   - Names of private, protected, internal and protected internal fields and properties: _camelCase.
   - Naming convention is unaffected by modifiers such as const, static, readonly, etc.
   - bool properties consist of questions that begin with Is, Has, Did etc.. when possible
   - For casing, a “word” is anything written without internal spaces, including acronyms. For example, MyRpc instead of MyRPC.
   - Names of interfaces start with I, e.g. IInterface.
   - We avoid `this.` and `var` unless absolutely necessary.
   - We always specify the visibility, even if it's the default (e.g. private string _foo not string _foo). Visibility should be the first modifier (e.g. public abstract not abstract public).


Files

    Filenames and directory names are PascalCase, e.g. MyFile.cs.
    Where possible the file name should be the same as the name of the main class in the file, e.g. MyClass.cs.
    In general, prefer one core class per file.

Organization
    Namespace using declarations go at the top, before any namespaces. using import order is alphabetical, apart from System imports which always go first.
    Class member ordering:
        Group class members in the following order:
            Nested classes, enums, delegates and events.
            Static, const and readonly fields.
            Fields and properties.
            Constructors and finalizers.
            Methods.
    in general favour this ordering:
            Public.
            Internal.
            Protected internal.
            Protected.
            Private.

Unity Specific 



### Example File:

``MyClass.cs:``

```C#
using System;                                       // `using` goes at the top, outside the namespace

namespace MyNamespace                               // Namespaces are PascalCase.
{
  public interface IMyInterface
  {                                                 // Interfaces start with 'I'
    public int Calculate(float value, float exp);   // Methods are PascalCase and space after comma.
  }

  public enum MyEnum                                // Enumerations are PascalCase.
  {                             
    Yes,                                            // Enumerators are PascalCase.
    No,
  }

  public class MyClass                              // Classes are PascalCase.
  {
    public event Action OnDoorOpened;               //prefixes events with on , and name after their desired behaviour
    public static int NumTimesCalled = 0;           // Field initializers are encouraged.                 
    public int Foo = 0;                             // Public member variables are PascalCase.
    public bool isCounting = false;                 // boolen starts with is , has 
    private Results _results;                       // Private member variables are _camelCase.   
    private const int _bar = 100;                   // const does not affect naming convention
   
    public int CalculateValue(int mulNumber)
    {     
      var resultValue = Foo * mulNumber;            // Local variables are camelCase.
      NumTimesCalled++;
      Foo += _bar;
      return resultValue;
    }

    // If aligning argument lines with the first argument doesn't fit, or is difficult to
    // read, wrap all arguments on new lines with a 4 space indent.
    void AnotherLongFunctionNameThatCausesLineWrappingProblems(
        int longArgumentName, int longArgumentName2, int longArgumentName3) {}
  }
}

```

``ObservableLinkedList`1.ObservableLinkedListNode.cs:``

```C#
using System;

namespace System.Collections.Generics
{
    partial class ObservableLinkedList<T>
    {
        public class ObservableLinkedListNode
        {
            private readonly ObservableLinkedList<T> _parent;
            private readonly T _value;

            internal ObservableLinkedListNode(ObservableLinkedList<T> parent, T value)
            {
                Debug.Assert(parent != null);

                _parent = parent;
                _value = value;
            }

            public T Value
            {
                get { return _value; }
            }
        }

        ...
    }
}
```

For other languages, our current best guidance is consistency. When editing files, keep new code and changes consistent with the style in the files. For new files, it should conform to the style for that component. If there is a completely new component, anything that is reasonably broadly accepted is fine. For script files, please refer to the scripting blog for [tips](https://devblogs.microsoft.com/scripting/tag/powertip) and [best practices](https://devblogs.microsoft.com/scripting/tag/best-practices).
