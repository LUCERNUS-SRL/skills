# Report Format

The report goes to the user as one message once Group is done.

## Template

````md
## Recon: {sharpened question}

- **Stack**: {stack}
- **Constraints**: {constraints}
- **Repos read**: {N}
- **Consensus**: {approach} ({k} of {N}) | none

### 1. {Approach name}

{How it works, in one or two sentences.}
Used by: [{owner/repo}](https://github.com/{owner}/{repo}), [{owner/repo}](https://github.com/{owner}/{repo})

```{language}
{the excerpt from one of its repos' notes}
```

Why: {the reason one of its repos' docs give}
Versus the project: {what the project does today, and what would change}
Fits when {condition in the project}; costs {cost in the project}.

### 2. {Approach name}

...
````

## Example

````md
## Recon: How do large ASP.NET Core monoliths keep their modules apart?

- **Stack**: .NET 8, ASP.NET Core
- **Constraints**: several teams, one database, one deployable
- **Repos read**: 8
- **Consensus**: Module projects (5 of 8)

### 1. Module projects

Each module is a set of class libraries with its own DbContext and schema. The host calls one registration per module, and modules talk only through a public contracts project.
Used by: [acme/ledger](https://github.com/acme/ledger), [globex/fleet](https://github.com/globex/fleet), [initech/payroll](https://github.com/initech/payroll), [umbrella/clinic](https://github.com/umbrella/clinic), [hooli/ads](https://github.com/hooli/ads)

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

Why: one schema per module lets a team change its tables without a cross-team review.
Versus the project: the project has one `AppDbContext` and one `Services/` folder; each module would get its own projects, DbContext and schema.
Fits when teams own modules end to end; costs splitting `AppDbContext`, which every feature uses today.

### 2. Vertical slices

One project, one folder per feature, one file per use case holding its request, handler and endpoint, dispatched through MediatR.
Used by: [stark/forge](https://github.com/stark/forge), [wayne/vault](https://github.com/wayne/vault), [oscorp/labs](https://github.com/oscorp/labs)

```
src/Web/Features/
├── Orders/
│   ├── CreateOrder.cs
│   ├── GetOrder.cs
│   └── CancelOrder.cs
└── Invoices/
    └── IssueInvoice.cs
```

Versus the project: the project groups code by layer (`Controllers/`, `Services/`, `Repositories/`); each use case would move into one file under its feature.
Fits when changes land one use case at a time; costs the module boundary: any feature can still read another's tables.
````

## Rules

- **Why** is left out when no note behind the approach gives one.
- **Versus the project** is left out when the project has no code for the question yet.
- **Fits when and costs** speak in the project's terms: its constraints and its code.
- **Repos read** adds `(thin survey)` when fewer than 5 repos were read.
