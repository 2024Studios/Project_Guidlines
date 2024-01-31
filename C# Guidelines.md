C# Coding Style
===============

Based on :
- https://google.github.io/styleguide/csharp-style.html
- https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md

Tools :
 - Use CodeMaid to help beatify and clean up code
 - https://marketplace.visualstudio.com/items?itemName=SteveCadwallader.CodeMaid

Rule summary:

Code
   - Names of classes, methods, enumerations, public fields, public properties, namespaces: PascalCase.
   - Names of local variables, parameters: camelCase.
   - Names of private, protected, internal and protected internal fields and properties: _camelCase.
   - Naming convention is unaffected by modifiers such as const, static, readonly, etc.
   - bool properties consist of questions that begin with `Is`, `Has`, `Did` etc.. when possible
   - For casing, a “word” is anything written without internal spaces, including acronyms. For example, MyRpc instead of MyRPC.
   - Names of interfaces start with I, e.g. IInterface.
   - Avoid `this.` and `var` unless necessary.
   - Always specify the visibility, even if it's the default (e.g. private string _foo not string _foo). Visibility should be the first modifier (e.g. public abstract not abstract public).
   - When calling a delegate or events, use Invoke() and use the null conditional operator - e.g. SomeDelegate?.Invoke()

Unity Specific 
   - Favour the use of `[SerializeField]` and also write `private` along with it for example `[SerializeField] private int _mySerializedField;`
   - Favour the use of Auto-Implemented Properties where possible for example `public int PageNumber { get; set; }` instead of having an extra private variable pageNumber
   - Prefix any method that will be called from the editor when a button is clicked with `OnButtonClick_FunctionName`
      - Avoid having other methods call this `OnButtonClick_FunctionName` method
      - if for example, the button will purchase an item, then have this `OnButtonClick_FunctionName` call another method to do the actual purchase
      - the logic inside `OnButtonClick_FunctionName` should handle any button feedback and call any corresponding methods related to player input
   - Suffix any event listener method with `Callback` for `example DoorOpenedCallback`
   - One possible and applicable avoid simple one-word method names for example you can replace `Interact()` with `StartInteraction`
   - Name variables based on their types for example a variable of type `LevelManager` should be named `_levelManager` or `LevelManagerRef` or something similar
   - Aim for Zero Garbage collection
      - Check this website for more info https://docs.unity3d.com/Manual/performance-garbage-collection-best-practices.html
      - Avoid LINQ as possible, LINQ causes a lot of GC, and almost always all the cases can be done with a simple for-loop
      - Use a unity method that has NonAlloc like `RaycastNonAlloc`
   - Always cache components and reference at the start or use lazy initialization
         - For example, don't do GetComponent at update or similar methods that will be called a lot, get the components you need at Start instead
         - Lazy initialization is where you get the reference you need once you need it, and then never check for it again.

     
Files
   - Filenames and directory names are PascalCase, e.g. MyFile.cs.
   - In general, prefer one core class per file.

 Organization
   - Namespace using declarations go at the top, before any namespaces. using import order is alphabetical, apart from System imports which always go first.
   - Class member ordering:
        Group class members in the following order:
            Nested classes, enums, delegates and events.
            Static, const and readonly fields.
            Fields and properties.
            Constructors and finalizers.
            Methods.
   - in general favour this ordering:
            Public.
            Internal.
            Protected internal.
            Protected.
            Private.





### Example File:

``MyClass.cs:``

```C#
using System;                                             // `using` goes at the top, outside the namespace

namespace MyNamespace                                     // Namespaces are PascalCase.
{
  public interface IMyInterface
  {                                                       // Interfaces start with 'I'
    public int Calculate(float value, float exp);         // Methods are PascalCase and space after comma.
  }

  public enum MyEnum                                      // Enumerations are PascalCase.
  {                             
    Yes,                                                  // Enumerators are PascalCase.
    No,
  }

  public class MyClass                                    // Classes are PascalCase.
  {
    public event Action OnResultChange;                   //prefixes events with on, and name after their desired behavior
    public static int NumTimesCalled = 0;                 // Field initializers are encouraged.                 
    public int Foo = 0;                                   // Public member variables are PascalCase.

    private bool _isCounting = false;                     // boolen starts with is , has
    private Results _results;                             // Private member variables are _camelCase.   
    private const int _bar = 100;                         // const does not affect naming convention
   
    public int CalculateValue(int mulNumber)
    {     
      int resultValue = Foo * mulNumber;                  // Local variables are camelCase.
      OnResultChange?.Invoke();                           // use Invoke to fire delegate and events
      return resultValue;
    }

    public void ResultChangeCallback(){}                  // Suffix the event listener method with “Callback”

    public bool IsNewPosition(Vector3 currentPosition)    // EXAMPLE: Methods ask a question when they return bool.             
    {
      return (transform.position == newPosition);
    }

    private void DoPrivateFunction() { }                  //ALWAYS specify the access modifier

    // If possible, wrap arguments by aligning newlines with the first argument.
    void AVeryLongFunctionNameThatCausesLineWrappingProblems(int longArgumentName,
                                                             int p1, int p2) {}

    // If aligning argument lines with the first argument doesn't fit, or is difficult to
    // read, wrap all arguments on new lines with a 4 space indent.
    private void AnotherLongFunctionNameThatCausesLineWrappingProblems(
        int longArgumentName, int longArgumentName2, int longArgumentName3) {}
  }
}

```

``MyUnityClass.cs:``

```C#
using System;
  public class MyUnityClass
  {
   [SerializeField] private int _mySerializedField;
   [field: SerializeField] public int foo { get; private set; }      // you can use serialize field on properties 
 
   private void Start();                                             // Don't use public access modifiers for MonoBehaviour-specific methods
   public void InflictDamage(float damage, bool isSpecialDamage) { } // EXAMPLE: Methods start with a verb.

   public OnButtonClick_StartGame()                                  // Method called from button click OnButtonClick
   {} 
  }
```

