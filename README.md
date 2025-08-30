# Blog_API_DotNet

## 1. Add project references (dependency rule)

- Application depends on Domain + SharedResources
- Infrastructure depends on Application, Domain, SharedResources
- Api depends on Application, SharedResources
- Tests reference the layers they test

```sh
# Application -> Domain, SharedResources
dotnet add  BlogApi.Application/BlogApi.Application.csproj reference  BlogApi.Domain/BlogApi.Domain.csproj  BlogApi.SharedResources/BlogApi.SharedResources.csproj

# Infrastructure -> Application, Domain, SharedResources
dotnet add  BlogApi.Infrastructure/BlogApi.Infrastructure.csproj reference  BlogApi.Application/BlogApi.Application.csproj  BlogApi.Domain/BlogApi.Domain.csproj  BlogApi.SharedResources/BlogApi.SharedResources.csproj

# Api -> Application, SharedResources
dotnet add  BlogApi.Api/BlogApi.Api.csproj reference  BlogApi.Application/BlogApi.Application.csproj  BlogApi.SharedResources/BlogApi.SharedResources.csproj

# Unit tests -> Application, Domain
dotnet add tests/BlogApi.UnitTests/BlogApi.UnitTests.csproj reference  BlogApi.Application/BlogApi.Application.csproj  BlogApi.Domain/BlogApi.Domain.csproj

# Integration tests -> Api (for WebApplicationFactory)
dotnet add tests/BlogApi.IntegrationTests/BlogApi.IntegrationTests.csproj reference  BlogApi.Api/BlogApi.Api.csproj
```

## 2. Recommended NuGet packages (install now)

- I’ll pick PostgreSQL as the primary DB (Npgsql) — we can switch to SQL Server later by replacing the EF provider packages.

```sh
# Infrastructure: EF Core + Postgres + Identity
dotnet add   BlogApi.Infrastructure package Microsoft.EntityFrameworkCore
dotnet add   BlogApi.Infrastructure package Npgsql.EntityFrameworkCore.PostgreSQL # Microsoft.EntityFrameworkCore.SqlServer (for SQL server)
dotnet add   BlogApi.Infrastructure package Microsoft.EntityFrameworkCore.Design
dotnet add   BlogApi.Infrastructure package Microsoft.AspNetCore.Identity.EntityFrameworkCore

# Application: validation + mapping fallback
dotnet add   BlogApi.Application package FluentValidation
dotnet add   BlogApi.Application package Mapster

# Api: auth, swagger, serilog, fluent validation integration, api versioning
dotnet add   BlogApi.Api package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add   BlogApi.Api package Swashbuckle.AspNetCore
dotnet add   BlogApi.Api package Serilog.AspNetCore
dotnet add   BlogApi.Api package Serilog.Sinks.Console
dotnet add   BlogApi.Api package FluentValidation.AspNetCore
dotnet add   BlogApi.Api package Microsoft.AspNetCore.Mvc.Versioning

# Tests
dotnet add tests/BlogApi.UnitTests/BlogApi.UnitTests.csproj package Moq
dotnet add tests/BlogApi.UnitTests/BlogApi.UnitTests.csproj package FluentAssertions
dotnet add tests/BlogApi.IntegrationTests/BlogApi.IntegrationTests.csproj package Microsoft.AspNetCore.Mvc.Testing
```

- Install EF CLI tooling (if you haven’t already):

```sh
dotnet tool install --global dotnet-ef
# or update if already installed:
dotnet tool update --global dotnet-ef
```

## 3. Helpful dev commands (run locally)

```sh
# run the API
dotnet run --project   BlogApi.Api

# add EF migration (once DbContext is implemented)
dotnet ef migrations add InitialCreate --project   BlogApi.Infrastructure --startup-project   BlogApi.Api --context BlogDbContext -o Migrations

# apply migration
dotnet ef database update --project   BlogApi.Infrastructure --startup-project   BlogApi.Api --context BlogDbContext

```
