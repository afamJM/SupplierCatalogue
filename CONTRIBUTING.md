# Contributing

When contributing to this project, please adhere to the following guidelines.

## Submodules

As we are using submodules, you should run the following line in your terminal after checking out the solution

```
git submodule init
git submodule update --remote --recursive
```

More information can be found on the [Submodules page in confluence](https://tech.immediate.co.uk/confluence/display/WCPP/Git+working+practices+for+sub+modules)

## Coding Standards

[Microsoft C# Coding Guidelines](https://msdn.microsoft.com/en-us/library/ff926074.aspx)

[Microsoft .net Core Coding Style Guidelines](https://github.com/dotnet/corefx/blob/master/Documentation/coding-guidelines/coding-style.md)

### Naming

Please adhere to the following convension on naming wherever possible:

* All names should be descriptive without being excessively long.
* Boolean properties intended as a flag to describe the state of an object should have 
  names starting with 'is' or 'has' and describes the meaning of **true** in the context 
  of the property in question.
* The format of names hould follow the generally accepted best practice for the language in use.


### Comments

All types, methods and properties should have a standard C# comment block immediately before
the declaration.  Skeleton comment blocks can be generated by typing '///' on a the line before
the declaration once the declaration has been coded.  The summary, parameter values, return value
and exceptions should be kept concise.  If more discussion is required about the function of 
something then the remarks should be used.
