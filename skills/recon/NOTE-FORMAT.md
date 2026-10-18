# Note Format

A note holds what one repo shows. Each sub-agent returns its repo's note.

## Template

````md
### {owner/repo}

Approach: {how the repo handles the question, in one line | not found: {what you searched}}
Why: {the reason the repo's own docs give | none}
Seen in:
- `{path}`: {what this file shows}
- `{path}`: {what this file shows}

```{language}
// {path}
{the excerpt, verbatim}
```
````

A repo a git command failed on gets a note of one line:

```md
### {owner/repo}

Failed: {the error}
```

## Example

````md
### acme/ledger

Approach: each module is a set of class libraries under `src/Modules/`, registered by one `Add{Module}Module` call, with its own DbContext and database schema.
Why: one schema per module lets a team change its tables without a cross-team review.
Seen in:
- `src/Modules/Invoicing/Invoicing.Infrastructure/InvoicingModule.cs`: the module's registration and DbContext.
- `src/Bootstrapper/Program.cs`: the host calls one registration per module.
- `docs/adr/0004-module-schemas.md`: the reason for one schema per module.

```csharp
// src/Modules/Invoicing/Invoicing.Infrastructure/InvoicingModule.cs
public static class InvoicingModule
{
    public static IServiceCollection AddInvoicingModule(
        this IServiceCollection services, IConfiguration config)
    {
        services.AddDbContext<InvoicingDbContext>(o =>
            o.UseSqlServer(config.GetConnectionString("Main"),
                sql => sql.MigrationsHistoryTable("__Migrations", "invoicing")));
        services.AddScoped<IInvoicingApi, InvoicingApi>();
        return services;
    }
}
```
````

```md
### northwind/depot

Failed: fatal: unable to access 'https://github.com/northwind/depot/': Could not resolve host: github.com
```

## Rules

- **Approach** describes what the repo's code does, never what the repo is known for.
- **Why** reads `none` unless the repo's own docs (README, ADRs, CONTRIBUTING, design notes) give the reason, and that doc goes under Seen in.
- **Seen in** lists every file the Approach and Why rest on, as paths from the repo root.
- **The excerpt** is the code that shows the approach best, at most 40 lines, or a folder tree from `git ls-tree`.
- **Not found** notes carry no Why, Seen in or excerpt.
- **A note** stays under 150 words, the excerpt aside.
