# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas that may cause runtime issues.

---

## 3. Run Unit Tests

Execute the test suite in `Bookstore.Domain.Tests` to verify that domain logic behaves correctly after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test results carefully. Any failing tests should be investigated to determine whether they indicate a regression introduced during the migration or a pre-existing issue.

---

## 4. Verify Data Layer Functionality

Since `Bookstore.Data` handles data access, verify the following:

- **Database provider compatibility**: Confirm that the Entity Framework Core (or whichever ORM is in use) provider targets the correct version compatible with the new .NET target framework.
- **Migrations**: If Entity Framework Core is used, ensure existing migrations are intact and can be applied:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj
```

- **Connection strings**: Confirm that connection strings in configuration files (`appsettings.json`, environment variables, etc.) are correctly configured for the target environment.

---

## 5. Validate the Web Application

Run the `Bookstore.Web` project locally to confirm the application starts and functions as expected:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following manually:

- Application starts without runtime exceptions.
- Key routes and pages load correctly.
- Any authentication or authorization middleware functions as expected.
- Static assets are served correctly.

Review application logs for any runtime warnings or errors that did not surface at build time.

---

## 6. Review the CDK Project

The `Bookstore.Cdk` project likely defines infrastructure. Review the following:

- Confirm that any .NET version references within the CDK stack definition match the new target framework of the migrated projects.
- Validate that Lambda function handlers or other compute references point to the correct runtime (e.g., `dotnet8` instead of `dotnet6` or `netcoreapp3.1`).

---

## 7. Check Target Framework Consistency

Verify that all projects in the solution target the same or compatible .NET version by inspecting each `.csproj` file:

```xml
<TargetFramework>net8.0</TargetFramework>
```

Inconsistent target frameworks across projects can cause subtle runtime issues even when the build succeeds.

---

## 8. Review Removed or Changed APIs

Cross-platform .NET removes certain Windows-specific APIs. Search the codebase for any usage of APIs that may have been stubbed or suppressed during transformation but could fail at runtime:

- `System.Web` references
- Windows Registry access
- Windows-specific file path assumptions (e.g., backslash separators)

Use the .NET Upgrade Assistant compatibility analyzer or the `Microsoft.DotNet.PlatformAbstractions` tooling to assist with this review if needed.