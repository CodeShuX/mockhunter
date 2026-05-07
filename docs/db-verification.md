# DB Verification

When you provide MockHunter with database access, Phase 4 (Trace Provenance) gets significantly more accurate. Without DB access, many values fall to UNKNOWN. With DB access, MockHunter can confirm REAL vs MOCK.

This is **optional**. The audit works without it — just less precisely.

## What MockHunter does with DB access

For every value the audit suspects might be REAL (matched to an API response from your own backend), it runs a read-only SELECT to verify the value exists in the expected table.

**Example:**
- UI shows "Total Revenue: $4,231"
- Network log shows GET /api/revenue → 200 with `{ "total": 4231 }`
- MockHunter runs: `SELECT SUM(amount) FROM invoices WHERE status='paid'`
- If result matches → REAL (cite table.column)
- If result doesn't match → MOCK (likely server-side seeded)
- If table doesn't exist → MOCK or BROKEN

## Safety rules

MockHunter will **never** run any of:
- `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `DROP`
- `CREATE`, `ALTER`, `GRANT`, `REVOKE`
- Stored procedures or functions that mutate state
- Anything in a transaction that isn't read-only

If the skill is unsure whether a query is read-only, it skips that verification and falls back to UNKNOWN. **Use a read-only DB user if at all possible.**

## How to provide access

MockHunter accepts the connection as a **shell command** — whatever works in your terminal. Examples by stack:

### PostgreSQL — direct

```bash
psql "postgres://reader:password@db.example.com:5432/myapp"
```

The skill will append `-c "SELECT ..."` to run individual queries.

**Recommended:** create a read-only user:

```sql
CREATE USER mockhunter_reader WITH PASSWORD 'redacted' LOGIN;
GRANT CONNECT ON DATABASE myapp TO mockhunter_reader;
GRANT USAGE ON SCHEMA public TO mockhunter_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mockhunter_reader;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mockhunter_reader;
```

Then provide:

```bash
psql "postgres://mockhunter_reader:redacted@db.example.com:5432/myapp"
```

### PostgreSQL — Docker

```bash
docker exec -i my-postgres psql -U postgres -d myapp
```

The `-i` is important so the skill can pipe queries via stdin.

### MySQL

```bash
mysql -h db.example.com -u reader -ppassword myapp
```

**Recommended:** read-only user

```sql
CREATE USER 'mockhunter_reader'@'%' IDENTIFIED BY 'redacted';
GRANT SELECT ON myapp.* TO 'mockhunter_reader'@'%';
FLUSH PRIVILEGES;
```

### Supabase — service role key

Supabase apps usually expose a REST endpoint. Use `curl`:

```bash
curl -s 'https://YOURPROJECT.supabase.co/rest/v1/TABLE?select=*&limit=5' \
  -H "apikey: YOUR_SERVICE_ROLE_KEY" \
  -H "Authorization: Bearer YOUR_SERVICE_ROLE_KEY"
```

The skill will substitute `TABLE` and add filters per-query.

**Better option:** create a read-only Postgres user in your Supabase project (Settings → Database → connection string with the read-only role) and use the `psql` form.

### Supabase — direct Postgres

```bash
psql "postgres://postgres.xxx:password@aws-0-region.pooler.supabase.com:6543/postgres"
```

### MongoDB

```bash
mongosh "mongodb://reader:password@cluster.mongodb.net/myapp"
```

The skill will run `db.collection.findOne(...)`-style queries.

### PlanetScale / TiDB

Use the MySQL form — they're MySQL-compatible.

### SQLite (local dev)

```bash
sqlite3 /path/to/local.db
```

### Cloudflare D1

```bash
wrangler d1 execute my-database --command="SELECT ..."
```

The skill will run `wrangler d1 execute` per query.

### Firebase Firestore

v0.1.0 does not support Firestore directly (no shell command for query). **Workaround:** run a small read-only Express endpoint locally that proxies queries, then point MockHunter at that. Roadmap: native Firestore support in v0.2+.

## Privacy and security

- **Never paste production credentials** unless you trust your local Claude Code environment
- **Use a read-only role** — even if you trust the audit, defense-in-depth matters
- **Use a staging or dev DB** when possible — same data shape, lower risk
- **The DB connection string is logged** in `mockhunter-trace.json` for debugging. Sanitize before sharing.
- **Queries are logged in the report** — this means your schema becomes visible. Sanitize before publishing.

## Without DB access

If you can't (or don't want to) provide DB access, MockHunter still works. Trade-offs:

| With DB | Without DB |
|---|---|
| REAL vs MOCK distinguishable | Many values fall to UNKNOWN |
| Specific cite ("Stripe → DB invoices.amount_total") | Generic cite ("Found in /api/revenue response") |
| Catches server-side seeded data | Cannot tell server-seeded from real |
| Confirms uniformity is genuine vs templated | Heuristics only |

For most vibe-code reality checks, no-DB mode is fine — HARDCODED, MOCK, LLM, and BROKEN values are still detected purely from network + DOM. DB access is the difference between "looks like real data" and "confirmed real data."

## Troubleshooting

### "DB verification failed: connection refused"

- Check the connection string works in a fresh terminal
- Check firewall / VPN — your local machine might not have access to the DB server
- For Docker: confirm the container is running (`docker ps`)

### "DB verification failed: permission denied"

- Read-only user might not have SELECT on the specific table
- Run: `GRANT SELECT ON your_table TO mockhunter_reader;`

### Queries time out

- Big tables: MockHunter adds `LIMIT 5` to verification queries by default
- If still slow, simplify by providing a connection that pre-filters (e.g., a view)

### "DB query log too verbose"

- Sanitize `mockhunter-trace.json` before sharing
- Or skip DB verification entirely for that audit
