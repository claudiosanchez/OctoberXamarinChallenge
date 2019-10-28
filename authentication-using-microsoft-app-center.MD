## AUTHENTICATION USING MICROSOFT APP CENTER

_This article is part of the [October Xamarin Best Practices Challenge](https://medium.com/@claudiosanchez/the-september-xamarin-best-practices-challenge-819e098c8314)_

In an earlier post we learned how to [Get Started with Microsoft App Center](http://ttirrell.com/2019/10/getting-started-with-microsoft-app-center/) to monitor analytics and crashes for your app. Now we will take a deeper dive into implementing authentication within your app.

We will assume that you already have accounts setup for Azure and App Center. You also should have at least one app configured in App Center.

#### Create an Azure Active Directory B2C Tenant

Sign in to the [Azure portal](https://portal.azure.com/).

Next click ‘Create a resource’ at the top left of the side menu. Search for ‘Active Directory B2C’ and then select ‘Create’. _Notice that the first 50,000 authentications per month are free._

![Appcenter create tenant](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-204041.jpg)

We want to select ‘Create a new Azure AD B2C Tenant’. Now enter an ‘Organization name’ and ‘Initial domain name’ and then click ‘Create’.

Wait for the Azure to finish creating the directory. Once it is completed click the link ‘Create new B2C Tenant or Link to existing Tenant’ toward the top of the page.

![Appcenter manage tenant](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-231644.jpg)

Now we want to choose ‘Link an existing Azure AD B2C Tenant to my Azure subscription’. Select the Tenant we just created, choose your Azure subscription and then click ‘Create new’ for the Resource group and enter a Name. Click ‘OK’. Now choose a ‘Resource group location’ and click ‘Create’.

![Appcenter link existing tenant](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-231804.jpg)

#### Register an Application in Azure AD B2C

You will need to switch to the directory that contains your Azure AD B2C tenant by clicking on the ‘Directory + subscription filter’ in the top menu.

![Appcenter register app](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-211028.jpg)

Search for Azure AD B2C and click on it. Click on ‘Applications’ under the ‘Manage’ section and click ‘+ Add’.

Type in a ‘Name’ and then toggle ‘Yes’ for ‘Include web app / web API’. We will come back to the ‘Reply URL’ section in a moment. For now enter api to the end of the ‘App ID URI (optional)’ field. Toggle ‘Yes’ for ‘Include native client’ and enter msal{appSecret}://auth for the ‘Custom Redirect URI’. Replace {appSecret}with the app secret for your App Center app. Add a separate uri for each app.

We will configure Microsoft Account for our identity provider. See [this link](https://docs.microsoft.com/en-us/appcenter/auth/getting-started#set-up-the-redirect-uri-on-your-identity-provider) if you would like to use a different identity provider.

Enter https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp for the “Reply URL’. Now click ‘Create’. Make note of the ‘Application ID’, we will need this later.

![Appcenter app properties](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-215557.jpg)

Click on the application you just created and then click ‘API access’ and ‘+Add’. Select the name of your application in the ‘Select API’ dropdown and ‘Access this app on behalf of the signed-in user (user_impersonation)’ in the ‘Select Scopes’ dropdown. Click ‘OK’

![Appcenter api access](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-221319.jpg)

Now we need to head over to ‘All services’ in the side menu and search for ‘App registrations’. Click on the app you previously created.

![Appcenter app registration](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-222348.jpg)

Then select ‘Certificates & secrets’ under the ‘Manage’ section. Click ‘+ New client secret’ and enter a description. Click ‘Add’. Make note of the application password shown in the Value column.

![Appcenter create secret](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-222558.jpg)

Search for ‘Azure AD B2C’ in the top search box. Next click on ‘Identity providers’ under the ‘Manage’ section. Click ‘Microsoft Account’ and enter a ‘Name’. Paste the values for ‘Client ID’ and ‘Client secret’ that we saved earlier. Click ‘Save’

![Appcenter identity providers](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-232651.jpg)

Finally click on ‘User flows (policies)’ under the ‘Policies’ section and click ‘+ New user flow’. Click the ‘Sign up and sign in’ link. Enter a Name and select the ‘Identity providers’ you want to use. ‘Multifactor authentication’ costs $0.03 USD per authentication. Click ‘Show more…’ under ‘User attributes and claims’ and select the attributes you want to “Collect” from the user and “Return” to your app. Click ‘OK’ and ‘Create’. Make note of the ‘Name’ you entered.

![Appcenter create user flow](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-230445.jpg)

#### Configure App Center

Sign in to your [App Center](https://appcenter.ms/) account.

Select the app you want to configure and choose ‘Auth’ from the side menu. Click ‘Connect your Azure Subscription’.

![Appcenter auth setup](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-232931.jpg)

Select your Azure subscription and click ‘Next’. Select your Azure AD B2C tenant and click ‘Next’. Select your Azure app and click ‘Next’. Select the scope and click ‘Next’. Manually add the name of the user flow you made note of above. Click the checkbox and click ‘Connect’.

![Appcenter custom policy](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-225020.jpg)

#### Add the App Center Auth SDK to Your App

Open Visual Studio and either create a new app or select an existing app.

Right click on your Solution and select ‘Manage NuGet Packages for Solution…’. Brows for ‘AppCenter’ and install both **Microsoft.AppCenter **and **Microsoft.AppCenter.Auth** for all your projects.

![Appcenter install sdk](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-27-233907.jpg)

For Xamarin.Forms apps you will want to add the following ‘using’ statements into our App.xaml.cs file.

using Microsoft.AppCenter;
using Microsoft.AppCenter.Auth;

_You will need to navigate to each of the apps you’ve created in App Center to find the ‘App secret’ for each one. Make sure to replace {Your App Secret} with the actual value for your application in each of the code blocks below._

We need to copy and paste the following into the OnStart() method within the same file.

AppCenter.Start("ios=**{Your iOS App secret here}**;" +
                  "android=**{Your Android App secret here}**",
                  typeof(Auth));

##### **Android**

For Android you must add the following element to the project’s **AndroidManifest.xml** file within the application tag.

&lt;activity android:name="com.microsoft.identity.client.BrowserTabActivity&gt;
     &lt;intent-filter&gt;
         &lt;action android:name="android.intent.action.VIEW" /&gt;
         &lt;category android:name="android.intent.category.DEFAULT" /&gt;
         &lt;category android:name="android.intent.category.BROWSABLE" /&gt;
         &lt;data
             android:host="auth"
             android:scheme="msal**{Your App Secret}**" /&gt;
     &lt;/intent-filter&gt;
&lt;/activity&gt; 

##### **iOS**

For iOS you will need to right click on the Info.plist file and select ‘View Code’. Copy and paste the following code:

&lt;key&gt;CFBundleURLTypes&lt;/key&gt;
&lt;array&gt;
     &lt;dict&gt;
         &lt;key&gt;CFBundleTypeRole&lt;/key&gt;
         &lt;string&gt;Editor&lt;/string&gt;
         &lt;key&gt;CFBundleURLName&lt;/key&gt;
         &lt;string&gt;$(PRODUCT_BUNDLE_IDENTIFIER)&lt;/string&gt;
         &lt;key&gt;CFBundleURLSchemes&lt;/key&gt;
         &lt;array&gt;
             &lt;string&gt;msal**{APP_SECRET}**&lt;/string&gt;
         &lt;/array&gt;
     &lt;/dict&gt;
&lt;/array&gt; 

Next open the project’s Entitlements.plist and check **Enable Keychain **under the Keychain Entitlement. Then click ‘&lt;_Add New_&gt; and type com.microsoft.adalcache in the form, then hit Enter and Save the file.

#### Add Code to Authenticate Users

Here is some sample code you can test in your XAML file:

&lt;StackLayout&gt;
     &lt;Label x:Name="loginText" Text="Please Login"
         HorizontalOptions="Center"
         VerticalOptions="CenterAndExpand" /&gt;
     &lt;Button x:Name="loginButton" Text="Login"
             Clicked="loginButton_Clicked"/&gt;
     &lt;Button x:Name="logoutButton" Text="Logout"
             Clicked="logoutButton_Clicked"/&gt;
 &lt;/StackLayout&gt;

In your loginButton clicked event you will want to call a method to SignIn. And in the logoutButton clicked eventy you will want to call a method to SignOut. Here is what that might look like.

private async void loginButton_Clicked(object sender, EventArgs e)  
{  
await SignInAsync();  
}

private void logoutButton_Clicked(object sender, EventArgs e)  
{  
SignOut();  
}

Now create your SignInAsync() method

async Task SignInAsync()
{
    try
    {
        // Sign-in succeeded.
        UserInformation userInfo = await Auth.SignInAsync();

        loginText.Text = userInfo.AccountId;
        loginButton.Text = "Welcome to Xamarin.Forms!";
    }         
    catch (Exception e)
    {
        // Do something with sign-in failure.
        loginText.Text = "Unable to Login";
        loginButton.Text = "Try Again";
    }
}

Finally create your SignOut() method

void SignOut()
{
     Auth.SignOut();

     loginText.Text = "You are now signed out";
     loginButton.Text = "Please Login";
}

You will need to use an actual device to test authentication. An emulator or simulator will not work. Additionally your iOS device needs to be provisioned first (see references below).

![Appcenter screenshots](http://ttirrell.com/wp-content/uploads/2019/10/Annotation-2019-10-28-003636-1024x603.jpg)

#### Useful Links

- [Azure Portal](https://portal.azure.com/)
- [Microsoft App Center](https://appcenter.ms/)
- [Getting Started with App Center Auth Service](https://docs.microsoft.com/en-us/appcenter/auth/getting-started) documentation
- [Create an Azure Active Directory B2C tenant](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant) documentation
- [Setting up your Identity Provider](https://docs.microsoft.com/en-us/appcenter/auth/getting-started#set-up-the-redirect-uri-on-your-identity-provider) documentation
- [Device provisioning for Xamarin.iOS](https://docs.microsoft.com/en-us/xamarin/ios/get-started/installation/device-provisioning/) documentation

##### **Find me:**

- [YouTube](https://www.youtube.com/user/ttirrell)
- [Twitter](http://twitter.com/TrevorTirrell)
- [GitHub](http://github.com/ttirrell)