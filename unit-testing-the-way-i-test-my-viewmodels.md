# October Xamarin Challenge

## Unit testing, the way I test my ViewModels

If you're a MacOS developer, I wrote some related posts with the tooling and the code coverage. Please, check them because maybe they are useful.

* [Unit testing tools using your MacOS terminal](https://mookiefumi.com/2019-10-20-unit-testing-tools-using-your-MacOS-terminal)
* [Unit testing tools using VSCode](https://mookiefumi.com/2019-10-23-unit-testing-tools-using-your-vscode).

> This article is part of the October Xamarin Best Practices Challenge, you can read about it in [Github of Claudio Sanchez](https://github.com/claudiosanchez/OctoberXamarinChallenge).

### Introduction

Nowadays the unit test is something required and we shouldn't negotiated if we have to do it or not, we've been applying new design patterns in order to test (*in this case*) our ViewModels, so we don't have more excuses. Are you testing your MVVM components like your ViewModels?

Everytime I speak about unit testing, I remember the acronym **F.I.R.S.T**. It should be the key of every unit test. Have you ever read something about it?

* **F**. Our unit test should be fast.
* **I**. Our unit test should be isolated.
* **R**. Our unit test should be repeatable (not depends on data).
* **S**. Our unit test should be self-validation.
* **T**. Our unit test should be thorough.

Please, check [this link](https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing) to read more about F.I.R.S.T.

### The example

I am going to use a LoginPage example with some use cases, it's a simple one (enough, i hope) and it will help us to show you how you can do it. This is our LoginPage.

![LoginPage](images/Screenshot2019-10-23_18.26.57.png)

What are the main parts of my ViewModels I usually cover with unit testing?

* The code that has *business logic o presentation logic*, for example, enable or not a button when depends on a property value.

    E.g.: The login button is not enable until the username and password have value.

* To ensure the navigation between ViewModels.

    E.g.: The login is valid so the app should navigate to the MainPage.

* To ensure that the services receive the expected parameters from the ViewModel.

    E.g.: We call the Login method (LoginService) with the right parameters (Username/ password properties of the ViewModel).

* To ensure the behaviour when an exception ocurrs.

    E.g.: How our ViewModels handle the exceptions? Wich is the behaviour if the API response with a 401 Http status code?

I have in my tool belt some nuget packages that helps me to write tests but they are not required so if you have another tools, use them!

### Tooling

* **XUnit**. A free unit testing tool. [Link](https://xunit.net)
* **Fluent Assertions**. A set of extensions methods that allow you to more naturally specify the expected assert. [Link](https://fluentassertions.com)
* **Moq**. A framework to work with test doubles. [Link](https://github.com/moq/moq4)

### Testing the use cases

These are the behaviours that I want to cover with the unit test in the LoginPageViewModel.

* The login button is enable if the email and password are entered.
* If the login is not successful we should advise that something went wrong.
* If the device doesn't have internet connection we should show an alert.
* If the login is valid the user will navigate to the main page.

> **Use case**. The login command is enable if the email and password are filled and here I usually check the positive case and the negative one.

```csharp
[Fact]
//I check if the LoginCommand can be executed (true)
public void Can_Execute_LoginCommand_If_Username_And_Password_Are_Not_Empty_And_Null()
{
    _sut.Username = "not-empty";
    _sut.Password = "not-empty";

    var result = _sut.LoginCommand.CanExecute(null);

    result.Should().BeTrue();
}

[Theory]
[InlineData("", "anyValue")]
[InlineData("anyValue", "")]
[InlineData(null, "anyValue")]
[InlineData("anyValue", null)]
[InlineData("", "")]
[InlineData(null, null)]
//I check if the LoginCommand can be executed (false)
//Theory/ InlineData are the way that XUnit can execute a test case with different parameters
public void Cant_Execute_LoginCommand_If_Username_Or_Password_Are_Null_Or_Empty(string username, string password)
{
    _sut.Username = username;
    _sut.Password = password;

    var result = _sut.LoginCommand.CanExecute(null);

    result.Should().BeFalse();
}
```

> **Use case**. If the login is not successful we should advise that something went wrong.

```csharp
[Fact]
public async Task Show_Toast_If_Login_Is_NotValid()
{
    //The login method is configured to returns Not Valid if the username contains "aaa"
    _sut.Username = "aaanother@username";
    _sut.Password = _loginRequest.Password;

    await _sut.LoginCommand.ExecuteAsync();

    UserDialogs.Verify(m => m.Toast("Invalid login, please try it again", null));
}
```

> **Use case**. If the device doesn't have internet connection we should show an alert.

```csharp
[Fact]
public async Task Show_Alert_If_Has_No_Internet_Connection()
{
    //Creating a stub of ConnectivityService to return false (HasConnection method)
    var connectivityService = new Mock<IConnectivityService>();
    connectivityService.Setup(m => m.HasConnection()).Returns(false);
    var loginService = new LoginService(new Mock<IAppContextService>().Object, connectivityService.Object);

    var sut = new LoginPageViewModel(CoreServices, loginService)
    {
        Username = _loginRequest.Username,
        Password = _loginRequest.Password
    };

    await sut.LoginCommand.ExecuteAsync();
    UserDialogs.Verify(m => m.AlertAsync("Check your internet connection", null, null, null));
}
```

> **Use case**. If the login is valid the user will navigate to the main page.

```csharp
[Fact]
public async Task NavigateTo_MyTabbedPage_If_Login_Is_Valid()
{
    _sut.Username = _loginRequest.Username;
    _sut.Password = _loginRequest.Password;

    await _sut.LoginCommand.ExecuteAsync();

    NavigationService.Verify(m => m.NavigateAsync($"/{nameof(MyTabbedPage)}"));
}
```

> The repo is in [Github](https://github.com/MookieFumi/Prism.101/tree/feature/prism) so you can check the full code there.
