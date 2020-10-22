# ASP.NET Core -- Effective Logging
This repo contains code that from the [Effective Logging in ASP.NET Core](https://app.pluralsight.com/library/courses/asp-dotnet-core-effective-logging) course on Pluralsight

# My Notes / Memory Joggers

**Filters** are used to create log entries globally across page and controller executions

See [TrackActionPerformanceFilter](https://github.com/teksidia/aspnetcore-effective-logging/blob/master/AspNetCore-Effective-Logging/BookClub.Infrastructure/Filters/TrackActionPerformanceFilter.cs)

```
services.AddMvc(options =>
{
    var builder = new AuthorizationPolicyBuilder().RequireAuthenticatedUser();
    options.Filters.Add(new AuthorizeFilter(builder.Build()));
    options.Filters.Add(typeof(TrackActionPerformanceFilter));
});
```
---
**Middleware** can help implement a global exception handling

See [UseApiExceptionHandler](https://github.com/teksidia/aspnetcore-effective-logging/blob/master/AspNetCore-Effective-Logging/BookClub.Infrastructure/Middleware/ApiExceptionMiddlewareExtensions.cs)

```
app.UseApiExceptionHandler(options =>
{
    options.AddResponseDetails = UpdateApiErrorResponse;
    options.DetermineLogLevel = DetermineLogLevel;
});
```
---
**Attributes** can allow a more granular approach

See [TrackPerformance](https://github.com/teksidia/aspnetcore-effective-logging/blob/master/AspNetCore-Effective-Logging/BookClub.Infrastructure/Attributes/TrackPerformanceAttribute.cs) ActionFilterAttribute

```
[TypeFilter(typeof(TrackPerformance))]
[Route("api/[controller]")]
```
---
**LifeCycle events** (via a base class) can also allow more granular control

See [BasePageModel](https://github.com/teksidia/aspnetcore-effective-logging/blob/master/AspNetCore-Effective-Logging/BookClub.Infrastructure/BaseClasses/BasePageModel.cs)

```
namespace BookClub.UI.Pages
{
    public class BookListModel : BasePageModel
    {
```


Level and Category/SourceContext are typically used to apply filters to logging

Scopes add contextual information to log entries across a single logical operation that might span class boundaries

Log levels allow you to categorise and filter based on urgency (fatal, error, warning, info, debug, trace etc)

To create logs: Serilog, NLog

To consume logs: Seq, ELK, App Insights

ILogger is injected via DI. But Serilog uses static Log.

To improve **logger performance**, use [LoggerMessage.Define](https://github.com/teksidia/aspnetcore-effective-logging/blob/master/AspNetCore-Effective-Logging/BookClub.Infrastructure/LogMessages.cs)

# Info

It includes references to BOTH Serilog and NLog and various commits will change the logging framework from one to the other.

A couple of key recent updates have been made that might be of interest:

* Updated the code base from .NET Core 2.2 to .NET Core 3.1 - and this meant updating Swashbuckle and some other NuGet packages as well.  To see the changes made to support this change, review this commit: https://github.com/dahlsailrunner/aspnetcore-effective-logging/commit/238ea8035a7852335c1cf320879eecffc1f9d39e
* Updated the code to support changes within the public [Demo IdentityServer4 instance](https://demo.identityserver.io).  The changes there were made to reflect recommended practices for flows.  To see the changes required in these applications, have a look at this commit: https://github.com/dahlsailrunner/aspnetcore-effective-logging/commit/6aa84a6a1cbdd6012cbab559a81ad20bab73b237

## Approach
Since I needed to be able to switch from one logging framework to another, I took an approach with this set of applications to tap primarily into the `Microsoft.Logging.Extensions` functionality rather than use Serilog Enrichers.  

## Getting Started
There are two projects that should be set to run in this repo: 
* BookClub.UI (this is the user interface - and it makes calls to the API project)
* BookClub.API (this is the API which has all of the database interactions)

The API projects `appsettings.json` file has a connection string that looks for a `BookClub` database on a local SQLExpress instance.  If you don't have one `BookClub` database:
* **Optional**: [Download Microsoft® SQL Server® 2017 Express](https://www.microsoft.com/en-us/download/details.aspx?id=55994) and [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&view=sql-server-ver15).
* Create a database called BookClub in some instance of SQL Server. Express is fine, so is the Docker version.
* Use the 4 files in the `BookClub.Data/Schema` folder to set up the `BookClub` database:
  * Create the `Book` table by running the SQL in `Book.sql`
  * Insert a couple of rows by running the SQL in `InitialData.sql`
  * Create stored procedures by running the SQL in both the `GetAllBooks.sql` and `InsertBook.sql` (each creates a proc)
* Update the connection string in `BookClub.API/appsettings.json` to point to the database.

Run the solution!  :)

## Exploring
There aren't that many pages / API methods here, and the best way to really explore what's going on is to simply try some of the pages and look at the code inside them, along with setting some breakpoints both in the UI and the API code.

