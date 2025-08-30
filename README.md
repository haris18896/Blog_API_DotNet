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

## 4. Project Structure

```sh
BlogApi/
│── src/
│   ├── BlogApi.Api/                  # API Layer (Controllers, DI, Config)
│   │   ├── Controllers/              # API endpoints
│   │   │   ├── AuthController.cs
│   │   │   ├── PostsController.cs
│   │   │   ├── CommentsController.cs
│   │   │   ├── LikesController.cs
│   │   │   └── BookmarksController.cs
│   │   │
│   │   ├── Middlewares/              # Custom middleware
│   │   │   ├── ErrorHandlingMiddleware.cs
│   │   │   └── LoggingMiddleware.cs
│   │   │
│   │   ├── Extensions/               # Extension methods for DI, Config
│   │   │   ├── ServiceCollectionExtensions.cs
│   │   │   └── ApplicationBuilderExtensions.cs
│   │   │
│   │   ├── Filters/                  # API filters (optional)
│   │   │   └── ValidateModelFilter.cs
│   │   │
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   └── appsettings.Development.json
│   │
│   ├── BlogApi.Application/          # Application Layer (Business logic)
│   │   ├── Interfaces/               # Abstractions
│   │   │   ├── IAuthService.cs
│   │   │   ├── IPostService.cs
│   │   │   ├── ICommentService.cs
│   │   │   ├── ILikeService.cs
│   │   │   └── IBookmarkService.cs
│   │   │
│   │   ├── Services/                 # Business logic implementations
│   │   │   ├── AuthService.cs
│   │   │   ├── PostService.cs
│   │   │   ├── CommentService.cs
│   │   │   ├── LikeService.cs
│   │   │   └── BookmarkService.cs
│   │   │
│   │   ├── DTOs/                     # Data Transfer Objects
│   │   │   ├── Auth/
│   │   │   │   ├── LoginRequest.cs
│   │   │   │   ├── RegisterRequest.cs
│   │   │   │   └── AuthResponse.cs
│   │   │   ├── Posts/
│   │   │   │   ├── CreatePostRequest.cs
│   │   │   │   ├── PostResponse.cs
│   │   │   │   └── UpdatePostRequest.cs
│   │   │   ├── Comments/
│   │   │   │   ├── CommentRequest.cs
│   │   │   │   └── CommentResponse.cs
│   │   │   └── Common/
│   │   │       ├── ApiResponse.cs
│   │   │       └── PaginationResponse.cs
│   │   │
│   │   ├── Validators/               # FluentValidation rules
│   │   │   ├── RegisterRequestValidator.cs
│   │   │   ├── LoginRequestValidator.cs
│   │   │   └── PostRequestValidator.cs
│   │   │
│   │   └── Mappings/                 # AutoMapper profiles
│   │       └── MappingProfile.cs
│   │
│   ├── BlogApi.Domain/               # Domain Layer (Entities, Enums, Constants)
│   │   ├── Entities/
│   │   │   ├── User.cs
│   │   │   ├── Post.cs
│   │   │   ├── Comment.cs
│   │   │   ├── Like.cs
│   │   │   └── Bookmark.cs
│   │   │
│   │   ├── Enums/
│   │   │   └── UserRoles.cs
│   │   │
│   │   └── Constants/
│   │       └── ErrorMessages.cs
│   │
│   ├── BlogApi.Infrastructure/       # Infra Layer (EF Core, Repos, JWT, etc.)
│   │   ├── Data/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   └── DbSeeder.cs
│   │   │
│   │   ├── Repositories/             # Data access
│   │   │   ├── GenericRepository.cs
│   │   │   ├── PostRepository.cs
│   │   │   └── UserRepository.cs
│   │   │
│   │   ├── Identity/                 # JWT, Auth, Password hashing
│   │   │   ├── JwtTokenGenerator.cs
│   │   │   └── PasswordHasher.cs
│   │   │
│   │   ├── Configurations/           # EF Core entity configs
│   │   │   ├── UserConfiguration.cs
│   │   │   ├── PostConfiguration.cs
│   │   │   └── CommentConfiguration.cs
│   │   │
│   │   ├── Migrations/               # EF Core migrations
│   │   │
│   │   └── DependencyInjection.cs    # Infra DI setup
│   │
│   └── BlogApi.Shared/               # Shared utilities (Optional)
│       ├── Helpers/
│       │   ├── DateTimeHelper.cs
│       │   └── SlugGenerator.cs
│       └── Exceptions/
│           ├── ApiException.cs
│           └── ValidationException.cs
│
│── tests/                            # Unit & Integration tests
│   ├── BlogApi.Tests/
│   │   ├── Services/
│   │   │   ├── AuthServiceTests.cs
│   │   │   └── PostServiceTests.cs
│   │   └── Controllers/
│   │       └── PostsControllerTests.cs
│
│── docker-compose.yml                # Docker (API + DB + Redis later)
│── Dockerfile
│── README.md
│── .gitignore
```

## 5. Highest-Level Diagram

```mermaid
flowchart LR
    Client[Client Apps<br>(Web, Mobile, Postman)] --> API[API Layer<br>(Controllers, Minimal APIs)]
    API --> App[Application Layer<br>(Services, Business Logic)]
    App --> Infra[Infrastructure Layer<br>(Repositories, Identity, External Services)]
    Infra --> DB[(Database<br>SQL Server / EF Core)]
    Infra --> Ext[External Services<br>(e.g., Email, File Storage, Auth Providers)]

    %% Flow back
    DB --> Infra
    Infra --> App
    App --> API
    API --> Client

    %% Cross-cutting
    subgraph CrossCutting[Cross-Cutting Concerns]
        Logging[Serilog Logging]
        Auth[JWT Auth & Roles]
        Errors[Middleware: Error Handling]
        Docs[Swagger/OpenAPI]
    end

    API --- CrossCutting
    App --- CrossCutting
    Infra --- CrossCutting
```
