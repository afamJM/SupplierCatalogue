# TESTING

There are two types of automated testing that must be performed on all our code.  These are:

*  [Unit Testing](#unit-testing)
*  [Integration Testing](#integration-testion)

##  Unit Testing

Unit testing is the testing of individual units of code in isolation.  For our purposes this usually means testing individual classes.

To facilitate testing, units of code should be as de-coupled as possible.  This can be achieved by following [SOLID principals](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)).  Using SOLID principals facilitates the mocking of dependancies, allowing code units to be tested without also testing units that they are dependaent on.  To do this, we 'mock' the dependancies so that we can test the code unit without needing to use code units it is dependant on and to get guaranteed behaviour from the dependancy.  This allows us to identify issues in the code unit under test without having to worry about issues in other code unts.

Testing methods and tools will differ between languaages and environments.  The scenarios we need to considder are:

*  [C# code running under .NET Core](#c#-code-running-under-.net-core)
*  [Typescript code running in the browser](#typescript-code-running-in-the-browser)
*  [Typescript code runing under NodeJS](#typescript-code-runing-under-nodejs)

###  C# Code Running Under.NET Core

The testing of C# code running under .NET Core is performed using test classes defined in one or more test projects in the solution.  The tests are written using the [XUnit Test Framework](https://xunit.github.io/docs/getting-started-dotnet-core.html).

XUnit test classes contain a number of test of 'Fact's, which are methods annotated with Xunit.FactAttribute.  When the tests are run, each Fact is executed in its own instance of the test class.  The order in which Facts are executed is undefined.

Test classes should kept as simple as possible by using fixture classes to manage the creation of test instances of classes for testing.  This enables the test class to concentrate exclusively on the tests and not about how to resolve dependancies of the class under test.

Any class to be tested that relies on dependancy injection should have a fixture  created to assist in testing it.  The fixture's constructor should create [mocked versions](#mocking-using-moq) of each of the the depedancies for the class under test. These mocked dependancies should then be used to create a new instance of the class.

Each mocked dependancy should be made available as a read only property of the fixture.  The property names should start with 'Mock' followed by the name of the mocked interface without the leading 'I'.

The instantiated instance of the class under test should be made available as a  read only property of the test fixture.

####  Mocking Using Moq

The [Moq Framework](https://github.com/Moq/moq4) is a framework that simplifies the creation of mock objects for testing purposes.  It enables the tester to create an instance of an interface without having to write a concrete class and to specify the behaviour of the mocked interface's methods when called with diferent parameters. It also allows the tester to check how often methods are called and with what parameters, raise and intercept events and other useful things.

####  Sharing a Fixture Instance Between Facts

Since a new test class is created for each test, instantiating an instance of 
the fxture in the test class constructor will ensure that each test has a fresh 
environment to test in.  This is desirable for most cases, but xunit also allows the same test fixture instance to be used for all tests in a test class.  

If the sharing of a fixture instance between tests is required, the test class should implement IClassFixture<*FixtureClass*> for the fixture class providing the test subject. The test class should then implment a constructor that takes a paramter of the fixture type, which is then saved in a memeber variable.  The xunit framework will automagically pass the same instance of the fixture to the constructor when creating the test class before each test.  If cleanup is requred between tests, the test class should implement IDisposable and do the cleanup in the Dispose method.

#### Putting it all together

A trivial example of all this could be as follows:

##### The Class Under Test

```C#
namespace Example
{
    /// <summary>
    /// An example class to test
    /// </summary>
    public class Example
    {
        private IDependancy dependancy;

        /// <summary>
        /// Initializes a new instance of the <see cref="Example" /> class.
        // </summary>
        public Example(IDependency dependancy)
        {
            this.dependancy = dependancy;
        }

        /// <summary>
        /// Get the enhanced value
        /// </summary>
        /// <param name="i">The input value</param>
        /// <returns>The enhanced value</returns>
        /// <remarks>
        /// The enhanced value is the input value plus the markup
        /// value from the IDependancy instance that was passed in the
        /// constructor.
        /// </remarks>
        public Example GetEnhancedValue(int i)
        {
            return i + this.dependancy.Markup
        }
    }
}
```

##### The Dependancy

```C#
namespace Example
{
    /// <summary>
    /// An example dependancy
    /// </summary>
    public interface IDependancy
    {
        /// <summary>
        /// The markup
        /// </summary>
        /// <value>
        /// The amount to markup
        /// </value>
        int Markup { get; }
    }
}
```

##### The Fixture

```C#
namespace Example.Tests.Fixtures
{
    using Moq;
    using Example;

    /// <summary>
    /// A test fixture for <see cref="Example"/>
    /// </summary>
    public class ExampleFixture
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="ExampleFixture"/> class.
        /// </summary>
        public ExampleFixture()
        {
            this.MockDependancy = new Mock<IDependancy>();
            this.Example = new Example(this.MockDependancy.Object);
        }

        /// <summary>
        /// Gets the mocked dependancy.
        /// </summary>
        /// <value>
        /// The mock dependancy.
        /// </value>
        public Mock<IDependancy> MockDependancy { get; private set; }

        /// <summary>
        /// Gets the example.
        /// </summary>
        /// <value>
        /// The example.
        /// </value>
        public Example Example { get; private set; }
    }
}
```

##### The Test Class

```C#
namespace Example.Tests
{
    using Moq;
    using Example.Tests.Fixtures

    /// <summary>
    /// A test class for the <see cref="ExampleFixture"/> class.
    /// </summary>
    public class ExampleTest
    {
        private ExampleFixture fixture;

        /// <summary>
        /// Initializes a new instance of the <see cref="ExampleTest"/> class.
        /// </summary>
        public ExampleTest()
        {
            this.fixture = ExampleFixture();
        }

        /// <summary>
        /// Should correctly add the markup to the suppied value.
        /// </summary>
        [Fact]
        public void ShouldEnhanceCorrectly()
        {
            this.fixture.MockDependancy.SetupGet(x => x.Markup).Return(3);
            var result = this.fixture.Example.GetEnhancedValue(4);
            this.fixture.MockDependancy.VerifyGet(x => x.Markup);
            Assert.Equal(12, result);
        }
    }
}
```

Note that at the time of testing, we don't even have to have an implelentation of the dependancy, we just need the interface.

The interaction between the Fact and the mock warrant a little explanation.  The line:
```C#
this.fixture.MockDependancy.SetupGet(x => x.Markup).Return(3);
```
tells Moq that any call to the getter for the Markup property on the mocked IDependancy should return the value 3.  The line:
```C#
this.fixture.MockDependancy.VerifyGet(x => x.Markup);
```
verifies that the getter the Markup property on the mocked IDependancy has been called.

###  Typescript Code Running in the Browser

**TBD**

###  Typescript Code Runing Under NodeJS

**TBD**

##  Integration Testing

**TBD**