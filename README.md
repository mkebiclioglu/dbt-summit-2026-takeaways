# dbt Summit 2026 — Takeaways

Notes from the two main-stage keynotes at dbt Summit 2026 (Las Vegas, Sept 15–18):

- **Day 1 — Keynote: Level Up** (Wed Sept 16, ~1h43m)
- **Day 2 — Community Keynote** (Thu Sept 17, ~1h06m)

Written from the session recordings' auto-generated captions, so a few product and people names may be slightly off. This is a summary, not a transcript. The recordings are on the [dbt Summit site](https://www.getdbt.com/dbt-summit/) (registration required).

See also: [dbt state, auto deferral and auto cloning](dbt-state-auto-cloning.md), a closer look at the new cloning behaviour.

---

## TL;DR

| Announcement | Status | What it is |
|---|---|---|
| **dbt v2** (formerly the Fusion engine) | GA | One Rust engine and one code base. The "Core vs Fusion" split is over, and both brands are retired in favour of just "dbt". |
| **v2 adapters** | GA: Redshift, BigQuery, Databricks, DuckDB, Snowflake. Beta: ClickHouse, Spark | Athena, Fabric, Postgres and more are coming "in the coming months". |
| **dbt state** (evolution of State-Aware Orchestration) | GA | Skips models whose code and upstream data haven't changed (quoted at 15–30% compute savings). Adds auto deferral and auto cloning across environments. Works on v1, v2, the dbt platform, Airflow and Snowflake dbt projects. |
| **Lake Compute** | Private beta | dbt's own DuckDB-based compute. Run individual models on it next to your warehouse, via Iceberg. |
| **dbt Charts** | Public beta | Open spec for charts and dashboards defined as YAML next to your models. |
| **dbt Wizard** | Public preview on the dbt platform (plus CLI) | dbt's analytics-engineering agent. Includes a new **explore mode** for business users. |
| **dbt desktop app** | Beta | A home for Wizard with parallel, isolated tasks and a visual review of changes. |
| **Native AI integrations** for the dbt MCP server | Launched | Connect once and dbt context shows up in tools your team already uses. |
| **Agent schema** | Spec (introduced June 2026) | A standard set of plain SQL tables in your warehouse where agents look for context. |
| **Fivetran context layer** | Private beta | Managed service that ingests structured and unstructured context, vectorises it and writes agent-schema tables. |
| **dbt information schema**, **checks**, **new dbt docs** | In v2 (both distributions) | Contracted project-metadata interface, SQL-based project quality checks, and docs rebuilt on DuckDB-WASM. |

**Next year:** dbt Summit 2027, Resorts World Las Vegas, Sept 13–16, 2027.

---

## Day 1 — Keynote: Level Up

### Framing (Tristan Handy, co-founder & president)
- The theme is "Level Up". The past year was when the industry went from "AI curious to truly AI pilled".
- Agents will become the **primary consumers** of metrics, tables and metadata, and that makes well-modelled data more valuable. Ramp's internal analytics agent drove a 20× jump in questions asked within weeks, and over 50× within six months.
- What's needed: infrastructure that gives agents context efficiently, eval infrastructure, and all of it **open and standards-based** to avoid lock-in.
- **Strategy post-Fivetran merger:** "reinventing the data platform with openness and AI at its core". It's positioned as both a complement to and an alternative to the vertically integrated platforms. The building blocks are Iceberg + Polaris for storage, Lake Compute, Fivetran connectors (the Connector SDK is MIT-licensed, with 100+ community connectors), dbt v2 for transformation, and then context, semantics, evals and analytics agents on top.

### dbt v2
- "The two engine era is over." Fusion becomes **dbt v2**, the first major version in about five years. The heart of it ships under **Apache 2.0**.
- 9,000+ weekly active projects are already on v2, including 36% of platform enterprise customers. Examples given: a 3,500-model project's parse time went from 3 min to 18 s, and an investment firm cut run times by more than 40%.
- **Spotify** (Tim Leonard) runs about 1,700 models on BigQuery and has always self-hosted on OSS. On v2, compiles take about 10 s versus 90+ s before. Their agentic workflows check code about 9× faster, and they estimate around 200k compute-hours saved once v2 is in production. Converting the project took "just a few days" with agents plus autofix.

### Live demo (Elias): v2, dbt state and Lake Compute
- **`dbt lint`**: a fast linter compatible with SQLFluff. It reuses your existing SQLFluff config and adds five dbt-specific rules, such as catching hard-coded references instead of `ref`/`source`.
- **Batch testing**: a flag that groups data tests into fewer warehouse scans. In the demo, 65 tests ran as 21 queries.
- **Local unit tests** (experimental): a `compute: local` config transpiles Snowflake/BigQuery SQL to DuckDB and runs unit tests on your laptop in milliseconds.
- **dbt state**: turned on with a managed-state flag in `dbt_project.yml`. Unchanged models are reused, and it does **auto deferral** and **auto cloning** for dev environments. → [details](dbt-state-auto-cloning.md)
- **Lake Compute**: set `table_format: iceberg` on the upstream model and `adapter: lake compute` on the model to move. That one model then runs on dbt's autoscaling DuckDB service while the rest stays on Snowflake. They called it the first public multi-adapter dbt invocation.

### Customer: OB Insurance (Tyson)
- Two-person data team with about 400 models, greenfield on v2, and roughly 99% uptime over the past year.
- Migration tip: use the **dbt agent skills** package and let a coding agent fix what it flags.
- dbt state saved them about 40% overall and up to 60% on their largest workflows, with tuning per model.
- About 10% of their models run on Lake Compute today, and they're targeting 80% by year-end.

### AI and context engineering (Tyler)
- Core message: "analytics engineering … is actually just context engineering."
- The recipe has four ingredients:
  1. **Meaning**: the semantic layer, with one definition per metric.
  2. **Reach**: the dbt MCP server (usage up 68× year over year) plus the new native AI integrations.
  3. **Discovery**: **agent schema**, plain SQL tables in a designated warehouse schema that any agent with SQL access can read.
  4. **Coverage**: unstructured context such as wikis, tickets and Slack. You model it like any other source.
- **Fivetran context layer** (private beta) handles that end to end. Early results showed agents using fewer tokens and making fewer calls than with off-the-shelf MCPs.
- **Okta** (Pooja) keeps agent context (markdown, YAML metadata, semantic definitions, agent definitions) in dbt and deploys it through PRs and CI. CI writes to agent schema, and the same context feeds Snowflake Cortex agents and Claude via MCP. The win: fixing a wrong answer is now a markdown change, with no redeploy.

### Product: Charts, Wizard and the desktop app (Sam Ferguson)
- **dbt Charts** (public beta): charts and dashboards as YAML in the repo, reviewed in the same PR/CI as the models. Pushing to Git makes the dashboard live with history and sharing.
- **dbt Wizard**: an agent built for analytics engineering. It investigates, edits, compiles, fixes and validates, and a human approves the result. It's in public preview on the dbt platform, with usage included in existing plans.
- **Explore mode**: business users ask plain-language questions grounded in the dbt project, and any follow-up work flows back to Wizard.
- **dbt desktop app** (beta): runs several Wizard tasks in parallel, isolated from each other, with visual lineage and diff review and comments on Wizard's output.
- Pattern: features that used to be platform-only are coming to the local CLI too.

### Partner panel: open data infrastructure
Panel: Russell Spitzer (Snowflake / Apache Iceberg PMC), Jordan Tigani (MotherDuck), Dipti Borkar (Microsoft OneLake / Fabric) and George Fraser (CEO, Fivetran + dbt Labs).
- **Iceberg read restrictions**: the Iceberg REST spec now lets a catalog send per-user access rules along with table metadata. It took about two years and 30–40 engineers across vendors. The spec is merged and implementation is underway.
- "May the best engine win": with data in an open format, each workload can pick its engine. MotherDuck sees Lake Compute as aimed at non-interactive transformation, while MotherDuck itself targets low-latency interactive use.
- **Semantic layers**: George argued a semantic model is now "necessary" and "sufficient" for AI over data, while Dipti said it's necessary but not sufficient. **OSI** (Open Semantic Interchange) is now an Apache incubating project with working code for converting semantic models between tools.
- **Evals** came up as a likely core data-engineering job, along with a call for an open eval standard for natural-language questions.
- Adoption numbers: about 7% of Fivetran customers land data in Iceberg first, and that share is growing. Microsoft reported 40k paying Fabric/OneLake customers, and dbt in Fabric (about 2k preview customers) is going GA at the end of the month.

### George Fraser on the merger
- He was surprised by how strongly dbt Core users assume the company has nothing for them. dbt state, the VS Code extension and Wizard are all meant to work for Core users.
- Actions so far: a more open license for the v2 Rust engine, and the VS Code extension's sign-in wall and user limit removed.
- He asked the community to hold the company accountable to "openness and interoperability" — "We are not trying to build a walled garden."

---

## Day 2 — Community Keynote

The keynote was staged as a retro video-game adventure by hosts Grace Goheen and Jeremy Cohen: they race the "dbt speed run", then leave dbt to "free their data" from a walled garden.

### dbt v2 distributions
- One code base, **two distributions**:
  - **`dbt`** (superset): free to use, includes the SQL-comprehension features (static analysis, linting, column-level lineage), with optional paid extras such as dbt state. It's still a CLI you can run locally or in production.
  - **`dbt oss`**: Apache-2.0 components only. It has no SQL-comprehension features, but it does "everything that dbt core v1 can do and much more".
- Which one to pick, if you're on Core today:
  - Use it for the CLI → `dbt`.
  - Use it because it's free → `dbt` (still free).
  - Need fully open source you can modify freely → `dbt oss`.
- The **"dbt Core" brand is being retired**, but **v1 is still maintained "for years"**. New features land in v2 only.

### Upgrade demo (v1.12 → v2)
1. On 1.12, parse with the **v2 parser flag** to surface compatibility errors. v2 parses strictly, so unknown config keys are errors.
2. Run **`dbt autofix deprecation`**, which moves custom configs under `meta`, among other fixes.
3. Switch to v2.

### New in v2 (both distributions)
- **dbt information schema**: a contracted metadata interface backed by new Parquet artifacts in `target/`. They're faster and richer than the JSON artifacts, and you can query them from inside the project with `dbt show`. Column-level lineage is included with strict static analysis (superset only).
- **Checks**: SQL assertions against project metadata (e.g. "all models have descriptions", "public models have owners"). They run first in `dbt build` and fail the build. Inspired by community tools like dbt-project-evaluator, dbt-checkpoint and dbt-bouncer.
- **New dbt docs**: rebuilt on DuckDB + WASM in the browser, so it's still a static site you can self-host. It has a new DAG view, column-level lineage and much faster search.

### Open standards tour
- **Apache Arrow / ADBC** (Matt Topol, Columnar): columnar database connectivity that avoids the row ↔ column conversions of ODBC/JDBC. **dbt v2 uses ADBC as the standard connectivity for all adapters.** Columnar also publishes `dbc`, an open-source driver manager.
- **DuckDB & Iceberg** (Hannes Mühleisen): DuckDB is now a top-5 dbt adapter. DuckLake is DuckDB's simpler alternative to Iceberg, but they have invested heavily in Iceberg support too. DuckDB Labs is joining AWS, and DuckDB stays open source under the DuckDB Foundation, led by its original authors.
- **OSI** (Chris Merrick, Omni co-founder, ex-RJMetrics where dbt began): an open standard for exchanging semantic definitions, so a metric defined once keeps its meaning in every tool.
- dbt Labs / Fivetran contribute to Arrow/ADBC, DuckDB, Iceberg and OSI, propose **agent schema** and **dbt Charts**, and steward **Great Expectations**.

### Community awards
There are 30 dbt Champions now, up from 9 founding Champions who started as "Igniters". Winners as captioned (spellings may be off):

| Award | Winner |
|---|---|
| Community Champion of the Year | Marcelo (second year in a row) |
| Ambassador of the Year | Bruno Lima |
| Meetup Organizer of the Year | Jeremy Chia |
| Analytics Engineer of the Year | Angela Alves Ferreira |
| Data Engineer of the Year | Karthik Rajasekaran |
| Data Analyst of the Year | Selim (France) |
| Technical Contributor of the Year | Daisuke Taniwaki |
| Design Partner of the Year | Sophie Lee |
| AI Innovator of the Year | Vince Faller |
| Commit Comedian of the Year | Ken Kwan |

---

## What this means for a team on dbt Core today

1. **Plan the v2 move.** Run the v2 parser flag on 1.12, then `dbt autofix`. Decide between `dbt` (free, more features) and `dbt oss` (pure Apache 2.0).
2. **Try dbt state.** It's GA, works on v1 and v2, and is the quickest cost win on stage (15–40%+). Read [auto cloning](dbt-state-auto-cloning.md) before turning it on in dev.
3. **Adopt checks and the new docs** as soon as you're on v2. They can likely replace some project-evaluator-style tooling.
4. **Watch Lake Compute** (private beta, needs Iceberg upstreams) if you have a few expensive, non-interactive models.
5. **Start treating context as code**: semantic layer metrics, agent schema, and context markdown in the repo, the way Okta does it.
