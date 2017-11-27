---
services: active-directory
platforms: dotnet
author: jmprieur
level: 200
client: .NET native (WPF)
service: ASP.NET Core 2.0
endpoint: AAD V1
---

# Calling a ASP.NET Core Web API from a WPF application using Azure AD

## About this sample
### Scenario
You expose a Web API and you want to protect it so that only authenticated user can access it. 

This sample presents a Web API running on ASP.NET Core 2.0, protected by Azure AD OAuth Bearer Authentication. The Web API is exercised by a .NET Desktop WPF application. 
The .Net application uses the Active Directory Authentication Library (ADAL.Net) to obtain a JWT access token through the [OAuth 2.0](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code) protocol. The access token is sent to the ASP.NET Core Web API, which authenticates the user using the ASP.NET JWT Bearer Authentication middleware.

### more information
For more information about how the protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](http://go.microsoft.com/fwlink/?LinkId=394414).

> This sample been updated to ASP.NET Core 2.0.  Looking for previous versions of this code sample? Check out the tags on the [releases](../../releases) GitHub page.

### User experience with this sample
The Web API (TodoListService) maintains an in-memory collection of to-do items per authenticated user. Several applications signed-in under the same identity share the to-do list.

The WPF application (TodoListClient) enables a user to:
- Sign in. The first time a user signs it, a consent screen is presented letting him consent for the application accessing the TodoList Service and the Azure Active Directory. When s/he has signed-in, the user sees the list of to-do items exposed by Web API for the signed-in identity
- add more to-do items (buy clicking on Add item).

Next time a user runs the application, the user is signed-in with the same identity as the application maintains a cache on disk. Users can clear the cache (which will also have the effect of signing them out)  
![TodoList Client](.\Readme\todolist-client.png)



## How To Run This Sample

### Pre-requisites
- Install .NET Core for Windows by following the instructions at [dot.net/core](https://dot.net/core), which will include Visual Studio 2017.
- An Internet connection
- An Azure Active Directory (Azure AD) tenant. For more information on how to get an Azure AD tenant, please see [How to get an Azure AD tenant](https://azure.microsoft.com/en-us/documentation/articles/active-directory-howto-tenant/) 
- A user account in your Azure AD tenant. This sample will not work with a Microsoft account (MSA, live account), so if you signed in to the Azure portal with a Microsoft personal account and have never created a user account in your directory before, you need to do that now (See [Quickstart: Add new users to Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/add-users-azure-active-directory)

### Step 1:  Clone or download this repository

From your shell or command line:

`git clone https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore.git`

### Step 3:  Register the sample with your Azure Active Directory tenant

There are two projects in this sample.  Each needs to be separately registered in your Azure AD tenant.

#### Register the TodoListService web API

1. Sign in to the [Azure portal](https://portal.azure.com).
1. On the top bar, click on your account and under the **Directory** list, choose the Active Directory tenant where you wish to register your application.
1. Click on **More Services** in the left hand nav, and choose **Azure Active Directory**.
1. Click on **App registrations** and choose **New application registration**.
1. Enter a friendly **Name** for the application, for example 'TodoListService' and select 'Web Application and/or Web API' as the **Application Type**. For the **sign-on URL**, enter the base URL for the sample, which is by default `https://localhost:44351`.  Click on **Create** to create the application.
1. While still in the Azure portal, choose your application, click on **Settings** and choose **Properties**.
1. Find the Application ID value and copy it to the clipboard.
1. From the Azure portal, note the following information: 
    - The Tenant domain: See the App ID URI base URL. For example: contoso.onmicrosoft.com 
    - The Tenant ID: See the Endpoints blade (button next to *New application registration*). Record the GUID from any of the endpoint URLs. For example: da41245a5-11b3-996c-00a8-4d99re19f292. Alternatively you can also find the Tenant ID in the Properties of the Azure Active Directory object (this is the value of the **Directory ID** property)
    - The Application ID (Client ID): See the Properties blade. For example: ba74781c2-53c2-442a-97c2-3d60re42f403

#### Register the TodoListClient app

1. Sign in to the [Azure portal](https://portal.azure.com).
1. On the top bar, click on your account and under the **Directory** list, choose the Active Directory tenant where you wish to register your application.
1. Click on **More Services** in the left hand nav, and choose **Azure Active Directory**.
1. Click on **App registrations** and choose **New application registration**.
1. Enter a friendly **Name** for the application, for example 'TodoListClient-DotNet' and select 'Native' as the **Application Type**. For the **Redirect URI**, enter `https://TodoListClient`. Click on **Create** to create the application.
1. While still in the Azure portal, choose your application, click on **Settings** and choose **Properties**.
1. Find the Application ID value and copy it to the clipboard.
1. Configure Permissions for your application - in the Settings menu, choose the 'Required permissions' section, click on **Add**, then **Select an API**, and type "TodoListService" in the text box. Then, click on  **Select Permissions** and select 'Access TodoListService'.

### Step 4:  Configure the sample to use your Azure AD tenant

#### Configure the TodoListService C# project

1. Open the solution in Visual Studio.
2. In the TodoListService project, open the `appsettings.json` file.
3. Find the `Domain` property and replace the value with your AAD tenant domain, e.g. contoso.onmicrosoft.com.
4. Find the `TenantId` property and replace the value with the Tenant ID you registered earlier, 
5. Find the `ClientId` property and replace the value with the Application ID (Client ID) property of the Service application, that you registered earlier.

#### Configure the TodoListClient C# project

1. In the TodoListClient project, open `App.config`.
2. Find the app key `ida:Tenant` and replace the value with your AAD Tenant ID (GUID). Alternatively you can also use your AAD tenant Name (e.g. contoso.onmicrosoft.com).
3. Find the app key `ida:ClientId` and replace the value with the ApplicationID (Client ID) for the TodoListClient from the Azure portal.
4. Find the app key `ida:RedirectUri` and replace the value with the Redirect URI for the TodoListClient from the Azure portal, for example `https://TodoListClient`.
5. Find the app key `todo:TodoListResourceId` and replace the value with the ApplicationID (Client ID) of the Service application (a GUID)
6. If you changed the default value, find the app key `todo:TodoListBaseAddress` and replace the value with the base address of the TodoListService project.


### Step 6:  Run the sample

Clean the solution, rebuild the solution, and run it.  You might want to go into the solution properties and set both projects as startup projects, with the service project starting first.

When you start the Web API, you will get an empty web page. This is expected.

Explore the sample by signing in into the TodoList client, adding items to the To Do list, removing the user account (Clearing the cache), and starting again.  As explained, if you stop the application without removing the user account, the next time you run the application you won't be prompted to sign-in again - that is the sample implements a persistent cache for ADAL, and remembers the tokens from the previous run.

NOTE: Remember, the To Do list is stored in memory in this TodoListService sample. Each time you run the TodoListService API, your To Do list will get emptied.



## How the code was created?
### Code for the service
The code for the service was created in the following way:
#### Create of the web api using the ASP.NET templates
```
md TodoListService
cd TodoListService
dotnet new webapi -au=SingleOrg
```
#### Add a model (TodoListItem) and modifying the controller.
In the TodoListService project, add a folder named `Models` and then a file named `TodoItem.cs` with the following content:

```CSharp
namespace TodoListService.Models
{
    public class TodoItem
    {
        public string Owner { get; set; }
        public string Title { get; set; }
    }
}
```

Under the `Controllers` folder, rename the file `ValuesController.cs` to `TodoListController.cs`.
Make sure the content is the following:
```CSharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using TodoListService.Models;

namespace TodoListService.Controllers
{
    [Authorize]
    [Route("api/[controller]")]
    public class TodoListController : Controller
    {
        static ConcurrentBag<TodoItem> todoStore = new ConcurrentBag<TodoItem>();

        [HttpGet]
        public IEnumerable<TodoItem> Get()
        {
            string owner = (User.FindFirst(ClaimTypes.NameIdentifier))?.Value;
            return todoStore.Where(t => t.Owner == owner).ToList();
        }

        [HttpPost]
        public void Post([FromBody]TodoItem Todo)
        {
            string owner = (User.FindFirst(ClaimTypes.NameIdentifier))?.Value;
            todoStore.Add(new TodoItem { Owner = owner, Title = Todo.Title });
        }
    }
}
```
This code gets the todo list items associated with their owner, which is the identity of the user using the Web API. It also adds todo list items associated to with the same user.
There is no persistance as this would be beyond the scope of this sample


#### Change the App URL
If you are using Visual Studio 2017
1. Edit the TodoListService's properties (right click on `TodoListService.csproj`, and choose **Properties**)
1. In the Debug tab:
    1. Check the **Launch browser** field to `https://localhost:44351/api/todolist`
    1. Change the **App URL** field to be `https://localhost:44351` as this is the URL registered in the Azure AD application representing our Web API.
    1. Check the **Enable SSL** field


## Related content
### Other documentation / samples
The scenarios involving Azure Active directory with ASP.NET Core are described in ASP.Net Core | Security | Authentication | [Azure Active Directory](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/azure-active-directory/) 
