# dbt state: auto deferral and auto cloning

A closer look at the "new clone" behaviour shown in the Day 1 keynote demo. It's part of **dbt state**, which went GA at dbt Summit 2026.

## What was shown

A new developer has an empty dev schema and edits one model (`dim_platforms`).

1. **Auto deferral.** `dbt run -s dim_platforms` builds only that model in dev. Its upstream `ref` resolves to **prod** automatically. Today you get this by passing a prod `manifest.json` with `--state` and `--defer`. With dbt state, "auto deferral is managed for you out of the box."
2. **Auto cloning.** `dbt run -s +dim_platforms` also selects the upstream `stg_platforms`, which was never built in dev. Instead of rebuilding it, dbt **reused** it. On stage this was described as dbt state supporting "auto cloning across environments when it recognizes a model you're trying to build in one environment has already been built and is up to date in another."

It's turned on with a managed-state flag in `dbt_project.yml`, which was `false` in the demo and then switched on.

## How it differs from `dbt clone`

| | `dbt clone` (since v1.6) | Auto cloning in dbt state |
|---|---|---|
| How it starts | You run the `dbt clone` command yourself | Happens automatically during a normal `dbt run` or `dbt build` |
| State it needs | You pass a prod `manifest.json` with `--state` | dbt manages state for you, so there are no manifest artifacts to pass around |
| What gets cloned | Whatever you select | Only models dbt detects as **already built and up to date** in another environment |
| Main use | Seeding a CI or dev schema, blue/green deploys | Avoiding rebuilds of unchanged upstreams while you develop |

## Related facts from the keynotes
- dbt state as a whole is quoted at **15–30% compute savings** by skipping models whose code and upstream data are unchanged. OB Insurance reported about 40%, and up to 60% on their largest workflows with tuning per model.
- It runs on **v1 and v2**, the dbt platform, Airflow, and natively in Snowflake dbt projects.
- In the Day 2 keynote, dbt state was named as an example of the **optional paid features** in the `dbt` distribution. It isn't part of `dbt oss`.

## Open questions (not covered on stage)
- Which warehouses do a true zero-copy clone here, and what happens on platforms without one? Probably a view or a full copy, but that's unconfirmed.
- How is "up to date" decided? Code hash, source freshness, or both?
- Cost and permissions: does dev need read or clone grants on prod objects?
- Licensing and pricing for teams on the free `dbt` CLI versus the dbt platform.

Check the dbt state docs before relying on this behaviour.
