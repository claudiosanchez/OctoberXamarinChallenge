# Crashing like a pro with App Center

## Accept your fate ##

Face it, your app is going to crash at some point. No matter how well you test it or how great the QA team is nor the 100% code coverage or how extensive the device suite for testing you have. There are always going to be edge cases you cannot predict and your app is going to crash at some point.

Once you accept this fact your job becomes a lot less stressful as you realize your duty from now on is making sure that, when it happens, you get the most context possible, in order to be able to start fixing the problems as they appear and not after getting an angry email from a final user.

## Picture this scenario

You and your team have been working on an app for the last 34 weeks and it is ready for production. Because your team is really smart you are using App Center for Continuous Integration and Continuous Delivery, that way the releases will go smoothly every time. You have been testing the app with a big set of different devices with different DPIs and specs, doing unit tests for the code with a lot of code coverage. Everything should work just fine, right? After all, nothing ever goes wrong in production, right?

A few weeks after the release you get a call from the boss. The app has low review scores, a lot of the negative reviews complaining about constant crashes and error messages, none of them with a concrete explanation of what the issue is or steps to reproduce. “That’s impossible, must be trolls or something, it works on all the devices we tested, and we tested on a lot of devices” – you think to yourself as you get the bad news. Everyone is sad, and you wonder: “How could we have prevented this from happening? How can we fix this?”

## Enter App Center Diagnostics service

App Center Diagnostics makes it easier for developers to monitor the health of their apps. With the App Center Diagnostics SDK, you can remove a lot of the guesswork when figuring out the reasons for app crashes and failings. The SDK collects relevant data and uploads it to App Center, making it available for the developer team to analyze.

## Error reporting like a pro

After the SDK is set and you have done a few crashes you can see the diagnostics report by going to the Diagnostics option from the App Center Dashboard and you will see something like this:

![https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-3.39.21-PM-1024x547.png.pagespeed.ic.oafTeCl8Jj.webp](https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-3.39.21-PM-1024x547.png.pagespeed.ic.oafTeCl8Jj.webp)

You can see the details of each crash report and it looks something like this:

![https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-2.00.02-PM-1024x678.png.pagespeed.ic.UId1r8eQ4U.webp](https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-2.00.02-PM-1024x678.png.pagespeed.ic.UId1r8eQ4U.webp)

Notice how the stack trace of the exception that was thrown is shown in detail, this is the best lead we can get to find the root of a specific problem.

You can see the detail of each individual error report by going to the Reports tab. Each error report includes some metadata from the device and optional data you can provide yourself when capturing the errors, these can give you a better look at how things were before the exception was launch. It looks something like this:

![https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-2.02.22-PM-1024x700.png.pagespeed.ic.rLo_g9NnPl.webp](https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-2.02.22-PM-1024x700.png.pagespeed.ic.rLo_g9NnPl.webp)

How to actually use the SDK

To setup you need to do the following:

1. Add this nuget package: Microsoft.Appcenter.Crashes

![https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-4.12.37-PM-1024x673.png.pagespeed.ic.y7ksKXdq0x.webp](https://blog.torib.io/wp-content/uploads/2019/10/xScreen-Shot-2019-10-12-at-4.12.37-PM-1024x673.png.pagespeed.ic.y7ksKXdq0x.webp)

2. Include the Crashes type in the App Center SDK start call

```C#
 Microsoft.AppCenter.AppCenter.Start("ios=<ios key>" +
 "android=<some key goes here>", typeof(Crashes));
```

By doing this you have crash report all set. That means if the app crashes from an unhandled exception you will get the information.

To explicitly log a an exception you need to call the TrackError method. It takes two parameters, an Exception object and a dictionary of strings for the optional parameters.

```C#
try
{

 //Do something that throws an exception

}
catch (Exceptionex)
{

 Microsoft.AppCenter.Crashes.Crashes.TrackError(ex, new Dictionary<string, string> {
         { "ReferenceNumber", ReferenceNumber },
         { "Result", "Record not found" }
     });
 }
```
The SDK will remember if the app crashed during the previous execution. With this information at hand you can show the user a custom message assuring them that, even though the app exploded, you got this, and everything will be ok.

To know if the app crashed during the previous execution call the HasCrashedInLastSessionAsync method

```C#
 bool appCrashed = await Crashes.HasCrashedInLastSessionAsync();

 if(appCrashed){

 DisplayAlert("Oh no!", "Something bad happened during the last session, we are sorry for that and we will look into it", "Ok");

 }
```

## Wrapping things up

Had the team configured the Crash SDK in the scenario I mentioned at the start of the article, they would have been able to monitor the errors, isolate the problems by platform/device and fix them before they became a bigger issue. They would also have been able to notify the user that yes, the app crashed, but the team got an error report, providing the user with a better sense of security.

And with that I hope you now have all the tools you need to get your app crashing like a pro with the App Center CrashSDK.

I hope you liked the article and that it was useful. If you liked anything about it please share it on social media and if you have any questions or comments feel free to contact me on twitter [@eatskolnikov](https://twitter.com/eatskolnikov) and have a good one.