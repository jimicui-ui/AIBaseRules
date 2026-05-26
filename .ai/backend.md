---
description: Specific guidelines for C# .NET Microservices and GCP cloud integrations. Activates on backend code modifications.
globs: backend/**/*.cs, **/Dockerfile, **/k8s/**/*.yaml
alwaysApply: false
---

# .NET Microservices & GCP Architecture Rules

## 1. Microservice Patterns
- **API Design:** Use modern Minimal APIs for lightweight routing, fallback to Controller pattern only if specifically requested.
- **Dependency Injection:** Explicitly register dependencies in `Program.cs` using the correct lifecycles (`AddTransient`, `AddScoped`, `AddSingleton`).
- **Asynchronous Flow:** Use `async/await` natively through the entire call stack. Never block threads using `.Result` or `.Wait()`.

## 2. GCP & Cloud Security
- **Configuration:** Never hardcode environment variables, connection strings, or cloud keys. Use `IConfiguration` mapped to secret management tools.
- **Observability:** Ensure structural logging is implemented using standard log levels (`LogInformation`, `LogError`) to cleanly stream logs into GCP Cloud Logging.
- **Containers:** Keep Dockerfiles multi-staged and optimized for minimal container image sizes.
