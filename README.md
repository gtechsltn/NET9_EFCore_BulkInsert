# NET9 EFCore BulkInsert

https://learn.microsoft.com/en-us/ef/core/miscellaneous/connection-strings?tabs=dotnet-core-cli

```
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

```
// Data/YourDbContext.cs
using Microsoft.EntityFrameworkCore;

namespace YourNamespace.Data
{
    public class YourDbContext : DbContext
    {
        public YourDbContext(DbContextOptions<YourDbContext> options)
            : base(options)
        {
        }

        public DbSet<Product> Products { get; set; } // DbSet for Product entity

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Configure the Product entity here if needed
            modelBuilder.Entity<Product>()
                .Property(p => p.Name)
                .IsRequired()
                .HasMaxLength(100);

            modelBuilder.Entity<Product>()
                .Property(p => p.Price)
                .HasColumnType("decimal(18,2)");
        }
    }
}
```

```
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

```
using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.EntityFrameworkCore;

public class ProductService
{
    private readonly YourDbContext _context;

    public ProductService(YourDbContext context)
    {
        _context = context;
    }

    public void BulkInsertProducts(List<Product> products)
    {
        const int batchSize = 10000; // Adjust batch size as needed
        int totalRecords = products.Count;

        for (int i = 0; i < totalRecords; i += batchSize)
        {
            var batch = products.Skip(i).Take(batchSize).ToList();
            _context.Products.AddRange(batch);

            // Save changes for each batch
            _context.SaveChanges();
            _context.ChangeTracker.Clear(); // Clear the change tracker to avoid memory issues
        }
    }
}
```

```
public void InsertData()
{
    var products = new List<Product>();

    for (int i = 0; i < 1000000; i++)
    {
        products.Add(new Product
        {
            Name = $"Product {i + 1}",
            Price = (decimal)(new Random().Next(1, 100) + new Random().NextDouble())
        });
    }

    using (var context = new YourDbContext())
    {
        var productService = new ProductService(context);
        productService.BulkInsertProducts(products);
    }
}
```

```
// Startup.cs
using Microsoft.EntityFrameworkCore;
using YourNamespace.Data;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Configure DbContext with SQL Server
        services.AddDbContext<YourDbContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

        // Other service configurations
        services.AddControllersWithViews();
    }

    // Other methods...
}
```

```
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=.;Initial Catalog=mssql;Integrated Security=SSPI;Connect Timeout=30;Pooling=True;Max Pool Size=10;Encrypt=True;TrustServerCertificate=True;MultipleActiveResultSets=true;"
  },
}
```

```
dotnet ef migrations add InitialCreate
```

```
dotnet ef database update
```
