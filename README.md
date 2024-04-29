# Prismals

Prismals is a CLI tool designed to facilitate the integration of PostgreSQL Row-Level Security (RLS) with Prisma Migrations. After creating a migration using `prisma migrate dev --create-only`, running `prismals` will automatically generate SQL to append RLS configurations to the latest migration file.

Here's an example of the SQL generated by Prisma Migrate:

```sql
-- CreateTable
CREATE TABLE "Company" (
    "id" TEXT NOT NULL,
    "name" TEXT NOT NULL,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "Company_pkey" PRIMARY KEY ("id")
);
```

Prismals will add the following RLS settings:

```sql
-- RLS Settings
ALTER TABLE "Company" ENABLE ROW LEVEL SECURITY;
ALTER TABLE "Company" FORCE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation_policy ON "Company" USING("id" = current_setting('app.company_id'));
CREATE POLICY bypass_rls_policy ON "Company" USING (current_setting('app.bypass_rls', TRUE)::text = 'on');
```

## Usage

Create a migration with Prisma Migrate and then append RLS configurations using Prismals:

```bash
# Create a migration with Prisma Migrate --create-only
npx prisma migrate dev --create-only --name migration
# Use Prismals to add RLS settings and create policies for the specified tables
npx @shoito/prismarls --schema=./prisma/schema.prisma --migrations=./prisma/migrations --currentSettingIsolation=app.company_id --currentSettingBypass=app.bypass_rls
```

## schema.prisma

Add a `/// @RLS` comment in your `schema.prisma` to specify tables and columns for which RLS should be configured.

```prisma
model Company {
  id        String   @id @default(cuid()) /// @RLS
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

If table or column names are specified using `map`, include these in the `@RLS` annotation:

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  company   Company? @relation(fields: [companyId], references: [id])
  companyId String?  @map("company_id") /// @RLS(table: "users", column: "company_id")
  @@map("users")
}
```

## Command Options

- `--schema`: Path to your Prisma `schema.prisma` file. Default is `./prisma/schema.prisma`.
- `--migrations`: Directory path containing migration files generated by Prisma Migrate. Default is `./prisma/migrations`.
- `--currentSettingIsolation`: Specify the current setting for RLS isolation. Default is `app.tenant_id`.
- `--currentUser`: Specify if using `current_user` with RLS. Default is `false`.
- `--currentSettingBypass`: Specify the current setting for RLS bypass. Default is `app.bypass_rls`.

## Prisma Limitations

- Settings from `prisma db pull` do not include RLS configurations.
- Deploying schemas with `prisma db push` does not apply RLS settings.

## License

MIT
