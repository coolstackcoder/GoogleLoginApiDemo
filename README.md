# Step-by-Step Guide: Creating a Google Login API Demo

## Prerequisites
- .NET 6.0 SDK or later
- Visual Studio 2022 or Visual Studio Code
- A Google Cloud Platform account for OAuth 2.0 credentials

## Step 1: Create a new Web API project

1. Open a terminal or command prompt.
2. Navigate to the directory where you want to create your project.
3. Run the following command:
   ```
   dotnet new webapi -n GoogleLoginApiDemo
   ```
4. Navigate into the new project directory:
   ```
   cd GoogleLoginApiDemo
   ```

## Step 2: Add necessary NuGet packages

1. Add the Google authentication package:
   ```
   dotnet add package Microsoft.AspNetCore.Authentication.Google
   ```

## Step 3: Configure Google OAuth

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project or select an existing one.
3. Navigate to "APIs & Services" > "Credentials".
4. Click "Create Credentials" and select "OAuth client ID".
5. Choose "Web application" as the application type.
6. Set the authorized redirect URI to `https://localhost:5001/signin-google` (adjust the port if necessary).
7. Note down the Client ID and Client Secret.

## Step 4: Update Program.cs

Replace the content of `Program.cs` with the following:

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.Google;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddAuthentication(options =>
    {
        options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = GoogleDefaults.AuthenticationScheme;
    })
    .AddCookie()
    .AddGoogle(options =>
    {
        options.ClientId = "YOUR_GOOGLE_CLIENT_ID";
        options.ClientSecret = "YOUR_GOOGLE_CLIENT_SECRET";
    });

builder.Services.AddAuthorization();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

Replace `YOUR_GOOGLE_CLIENT_ID` and `YOUR_GOOGLE_CLIENT_SECRET` with the values from Step 3.

## Step 5: Create AuthController

Create a new file `Controllers/AuthController.cs` with the following content:

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.Google;
using Microsoft.AspNetCore.Mvc;

namespace GoogleLoginApiDemo.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class AuthController : ControllerBase
    {
        [HttpGet("login")]
        public IActionResult Login()
        {
            return Challenge(new AuthenticationProperties { RedirectUri = "/api/auth/callback" },
                GoogleDefaults.AuthenticationScheme);
        }

        [HttpGet("callback")]
        public IActionResult LoginCallback()
        {
            return Ok(new { Message = "Login successful", User = User.Identity?.Name });
        }

        [HttpGet("logout")]
        public async Task<IActionResult> Logout()
        {
            await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            return Ok(new { Message = "Logout successful" });
        }

        [HttpGet("status")]
        public IActionResult Status()
        {
            return Ok(new { 
                IsAuthenticated = User.Identity?.IsAuthenticated ?? false,
                UserName = User.Identity?.Name
            });
        }
    }
}
```

## Step 6: Create SecureController

Create a new file `Controllers/SecureController.cs` with the following content:

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace GoogleLoginApiDemo.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    [Authorize]
    public class SecureController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get()
        {
            return Ok(new { Message = $"This is a secure endpoint. Welcome, {User.Identity?.Name}!" });
        }
    }
}
```

## Step 7: Create HelloController

Create a new file `Controllers/HelloController.cs` with the following content:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace GoogleLoginApiDemo.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class HelloController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get()
        {
            return Ok(new { Message = "Hello, World!" });
        }
    }
}
```

## Step 8: Run the application

1. In the terminal, run:
   ```
   dotnet run
   ```
2. The API will start, typically on `https://localhost:5001` (the port may vary).

## Step 9: Test the API

You can use tools like Postman or Swagger UI (available at `/swagger` when running in development mode) to test your API endpoints:

- GET `/api/auth/login`: Initiates the Google login process
- GET `/api/auth/status`: Checks the current authentication status
- GET `/api/secure`: Accesses a protected endpoint (requires authentication)
- GET `/api/hello`: Accesses a public endpoint
- GET `/api/auth/logout`: Logs out the current user

Note: For the Google login process to work correctly in a browser-based tool like Swagger UI, you may need to disable pop-up blocking for the site.

## Conclusion

You now have a functioning Google Login API demo. This API can be used as a backend for various types of applications, including single-page apps and mobile apps. Remember to handle the authentication flow appropriately on the client side when integrating with a front-end application.