# ClickHouse User Management

This project starts a local ClickHouse server with Docker Compose.

Start the service:

```bash
docker compose up -d
```

Open a ClickHouse SQL shell:

```bash
docker compose exec clickhouse clickhouse-client --user default --password
```

When prompted, enter the password from `.env`.

Run the SQL below as `default`, or as another user that already has the
`ACCESS MANAGEMENT` privilege.

## Add A New User

Create a user with a password:

```sql
CREATE USER IF NOT EXISTS app_user
IDENTIFIED WITH sha256_password BY 'change_me';
```

Limit the user to local connections only:

```sql
CREATE USER IF NOT EXISTS local_user
IDENTIFIED WITH sha256_password BY 'change_me'
HOST LOCAL;
```

Allow the user to connect from any host:

```sql
CREATE USER IF NOT EXISTS app_user
IDENTIFIED WITH sha256_password BY 'change_me'
HOST ANY;
```

Check users:

```sql
SHOW USERS;
```

## Set User Permissions

Grant read-only access to one database:

```sql
GRANT SELECT ON analytics.* TO app_user;
```

Grant read and write access to one database:

```sql
GRANT SELECT, INSERT ON analytics.* TO app_user;
```

Grant table-management permissions:

```sql
GRANT CREATE TABLE, ALTER TABLE, DROP TABLE ON analytics.* TO app_user;
```

Grant all permissions on one database:

```sql
GRANT ALL ON analytics.* TO app_user;
```

Check the permissions granted to a user:

```sql
SHOW GRANTS FOR app_user;
```

Revoke a permission:

```sql
REVOKE INSERT ON analytics.* FROM app_user;
```

## Create A User Group

ClickHouse uses roles for user groups. Create a role, grant permissions to the
role, then assign the role to users.

Create a read-only role:

```sql
CREATE ROLE IF NOT EXISTS analytics_readonly;
GRANT SELECT ON analytics.* TO analytics_readonly;
```

Assign the role to a user:

```sql
GRANT analytics_readonly TO app_user;
```

Make the role active by default when the user logs in:

```sql
SET DEFAULT ROLE analytics_readonly TO app_user;
```

Check roles:

```sql
SHOW ROLES;
SHOW GRANTS FOR analytics_readonly;
SHOW GRANTS FOR app_user;
```

Remove a role from a user:

```sql
REVOKE analytics_readonly FROM app_user;
```

Drop a role:

```sql
DROP ROLE IF EXISTS analytics_readonly;
```

## Common Role Examples

Read-only users:

```sql
CREATE ROLE IF NOT EXISTS readonly;
GRANT SELECT ON analytics.* TO readonly;
GRANT readonly TO app_user;
SET DEFAULT ROLE readonly TO app_user;
```

Data writers:

```sql
CREATE ROLE IF NOT EXISTS writer;
GRANT SELECT, INSERT ON analytics.* TO writer;
GRANT writer TO app_user;
SET DEFAULT ROLE writer TO app_user;
```

Database maintainers:

```sql
CREATE ROLE IF NOT EXISTS maintainer;
GRANT SELECT, INSERT, CREATE TABLE, ALTER TABLE, DROP TABLE ON analytics.* TO maintainer;
GRANT maintainer TO app_user;
SET DEFAULT ROLE maintainer TO app_user;
```

## Notes

- Replace `analytics` with your database name.
- Replace `app_user` with the actual username.
- Prefer roles over granting many permissions directly to individual users.
- Use `sha256_password` instead of `plaintext_password`.
- User and role data is stored under `/var/lib/clickhouse/access`, which is
  persisted by this compose setup through `/data/clickhouse/data`.

## References

- ClickHouse `CREATE USER`: https://clickhouse.com/docs/sql-reference/statements/create/user
- ClickHouse `GRANT`: https://clickhouse.com/docs/sql-reference/statements/grant
- ClickHouse `CREATE ROLE`: https://clickhouse.com/docs/sql-reference/statements/create/role
- ClickHouse `SET DEFAULT ROLE`: https://clickhouse.com/docs/sql-reference/statements/set-role
