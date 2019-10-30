USE DATA TO QUANTIFY YOUR APP'S SUCCESS
# Analytics Best Practices for Xamarin iOS & Android Apps
Quick Tips For Product Owners, Business Analysts & Devs of Native Apps as well

When you've worked on multiple published apps, you'll learn unique lessons while implementing solutions for each of the challenges. One of the apps I worked on, had thousands of active users and received so complaints about the inability to login that one could assume that no one was able to login. What was worse is that we had no way to quantify what percent of login attempts succeeded. Lo and behold: the power of analytics! With 2 extra lines of code, we were able to figure it out!

As part of the Xamarin October Challenge, where developers share best practices across Xamarin, this article goes over implementing Analytics in apps.

## Common Challenges
Developers commonly code what they are told to do, but Business is not certain of what data from apps can help them. They know they want "data" but there is no industry standard of mobile analytics. There are UI design guidelines set out by Apple (Human Interface Guidelines) and Android (Material Design), and Accessibility guidelines (WCAG) that we follow for platforms, but for Analytics, companies essentially try different solutions and just improve on it over time. That sounds inefficient, and it is why we need more unity from the community out there.

## Why Aren't The Default Analytics Enough
By default, Apple & Android do record app analytics that are accessible to Admin users of the accounts, as you can see in the pictures below:

Overview of some of the analytics captured by the Google Play StoreOverview of some of the analytics captured by the Apple App StoreBut none of the default analytics can tell:
- What's the most used feature in the app that we should put more resources into?
- How many people have had trouble signing in? Or submitting the form?
- Which  product in the promotion do people like most? Are people not adding it because of the price?
- What does the sales funnel look like? How many users drop at the Shopping Cart because of the Shipping Fees?
- When in the day/week do I get the most app traffic, so I can send a Push Notification?

## What Analytics Events should I Set?
Below is a short list of calls for a simple "Login page", and what purpose they serve:

### PageView: Login
PageView events are a must have for every page, and are called as soon as the page appears. These events help with knowing which pages are being used the most. Knowing how many users have visited the checkout page, shopping cart page and the product page can help your company understand the "sales funnel". When users use the app after analytics have been set for the Login page, the event will be recorded as PageView: Login.

### SelectAction: LoginButton
SelectAction events are triggered as soon buttons are pressed. So you can tell the number of times users attempt to perform certain action. You don't need to attach them to every button that is pressed in the app, but important functionality like Upload Picture Icon, or when the Login button is tapped. A sample event for tapping the Checkout button could look like this SelectAction: Checkout Icon.

### Result: LoginSuccess
You can't rely on the difference between the number of times the Login Button was pressed and the number of Home page views to find out the login failure rate. Users could be going back and forth between the different pages, and that number will skew the actual login attempts. So, for results from important functionality in the app like logging in, a Result event can be very useful. If the authentication was successful, just before trying to go to the next page, you would fire an event like this: Result: LoginSuccess. To find out the Login failure rate, you can just calculate the difference between the count of Result: LoginSuccess events and SelectAction: LoginButton. Result events are also useful for external dependencies, like when the app is using the camera/files to upload something.
### TerriblePerf: API: POST api/v3/login
There are some people who like to record the exact Performance of every API call, with the exact time in seconds. You end up with a lot of data that is not actionable. I prefer tracking API calls that take longer than a certain threshold. If the time taken to login takes longer than 5 seconds, I would trigger this event. I would suggest tweaking the timer based on your app's needs, but it's useful data to present when you are asked why the app is working slowly. You know exactly which API call is slow, and which database tables need refreshing. So, if the login api is slow, you will see a spike in these events `TerriblePerf: API: POST api/v3/login` and something needs to be done ASAP.

### BadPerf: API: GET api/v3/login
If API calls made by  your app is slow, but not terribly slow, BadPerf can help. So if the time taken to login takes longer than 2 seconds but less than 5, an event like this is fired BadPerf: API: POST api/v3/login. Just like TerriblePerf, I would add the API endpoint (api/v3/login) and the HTTP method (POST) in the event name.
Exception: GET api/v3/login: 500 Internal Server Error
Most apps contain external dependencies in the form of Backend service calls to RESTful servers. Occasionally, servers might be down while trying to connect and the backend returns data that does not make sense to the app that's expecting a normal response. Exception events can be useful for diagnosing issues in Web services that work well in QA, but do not scale will in Production. Usually instances like that causes crashes, so it's useful to at least be able to track such events so we can code around those instabilities, before the app crashes.


---

## Which Xamarin Analytics package should I choose?
For smaller apps, developers have more freedom to choose. There are several guides on how to actually add 3rd party Analytics packages. I personally recommend using MS App Center because:
- Google Analytics has changed it's APIs and packages multiple times, and App Center abstracts away and handles all the changes with Google and Android.
- MS App Center is able to combine the analytics to make Crash tracking and fixing easier as you can see below:


For bigger apps where developers have to implement a specific library, you can follow along these instructions and simply replace my function call with yours. 

## What Code Do Devs Write?
Tracking events using MSAppCenter is as simple as adding a line where needed. You just have to be creative about where to add that line to avoid redundant event counts.
### For PageView
From within a Page, you can get the name of the class, so instead of writing the function like this `Analytics.TrackEvent("PageView: Login");` for each page, you can just get the name of the class (LoginViewController/LoginFragment/LoginPage) using `this.GetType().Name` and then remove the name (Login) using an extension method SplitPascalCase that will convert your page name fromLoginPage to "Login Page", as shown below
```
public static string SplitPascalCase(this string value) 
{
    return Regex.Replace(value, @"(?<=[A-Za-z])(?=[A-Z][a-z])|(?<=[a-z0–9])(?=[0–9]?[A-Z])", " ").Trim();
}
```
For native iOS developers, you would place the following snippet just before the end of ViewDidAppear(the last ViewController lifecycle function): `Analytics.TrackEvent(this.GetType().Name.Replace("ViewController", string.Empty).SplitPascalCase());`. 
That function would add a PageView: Login event. You could also place the snippet in a BaseViewController.cs file that inherits from UIViewController and have all your ViewControllers inherit from BaseViewController instead of UIViewController. So, as soon as any ViewController appears, it automatically calls the analytics PageView function.

For native Android developers, you would place the TrackEvent function just before the end of OnResume(the last Fragment lifecycle function):
Analytics.TrackEvent(this.GetType().Name.Replace("Fragment", string.Empty).SplitPascalCase());. Similar to iOS, you can implement it just once for all your Fragments, by just adding this in the OnResume of a BaseFragment which all you Fragments inherit from.

For Xamarin.Forms developers, you would place the TrackEvent function just before the end of your Page constructor:
`Analytics.TrackEvent(this.GetType().Name.Replace("Page", string.Empty).SplitPascalCase());`.

### For SelectAction & Result
For SelectAction and Result events, you can just place the TrackEvent function in the event handler of the button or icon that is pressed. So you would have to edit `button.TouchUpInside` event in iOS, the `button.Click` event on Android and `button.Clicked` in Xamarin.Forms.

### For Exceptions


### For BadPerf & TerriblePerf
Since the web services layer is the same for iOS, Android or Xamarin.Forms, they are all implemented the same way. You need a Stopwatch object that will Start() & Stop() before and after the Web Service calls are made to the APIs. Then and we will monitor the Elapsed DateTime property for the number of seconds, and report it as Bad or Terrible as we want. You can add a dictionary to provide extra information about the exact seconds so you can view it in the report in case of a Crash.
```
var analyticsDict = new Dictionary<string, string>();
analyticsDict.Add("TotalSeconds", performanceCounter.TotalSeconds.ToString());
// Track Event only if the performance is very slow
if (performanceCounter.Seconds > 5)
   Microsoft.AppCenter.Analytics.Analytics
   .TrackEvent(string.Format("TerriblePerf: {0}: {1}: Greater than 9", category, eventName), analyticsDict);
else if (performanceCounter.Seconds > 2)
   Microsoft.AppCenter.Analytics.Analytics
   .TrackEvent(string.Format("BadPerf: {0}: {1}: Greater than 3", category, eventName), analyticsDict);
```
## What Does it All Look Like In Action?
MS App Center displaying analytics eventsI have attached some screenshots of what data I currently receive by descending order of count. I notice that the dashboard page is the most popular, but there's also one functionality that is really popular with a pretty broad funnel.


Searching for "login" events in MS App CenterSearching for just "Login" events, shows that 40% of the time, users weren't able to login for this app. And every successful login also took longer than 3 seconds. Definitely a lot of work needed here.

## Other Considerations
* Dependency Injection (DI) - Using DI is the best way to implement Analytics for an Enterprise App. It removes the need to redo code if the Analytics Service changes. You can simply register the different Analytics Services (Microsoft, Google, Adobe) you would like, and create a AppAnalytics static class that contains static functions that abstract away a service like MicrosoftAnalyticsService using the IAnalyticsService.
