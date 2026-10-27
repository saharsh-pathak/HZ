# Naming Strategies — Deep Reference

This document provides exhaustive naming transformation tables for the HZ skill.

## Generic → Domain-Specific Replacements

The key insight: AI detectors flag **high-probability token sequences**.
`result = process(data)` is the most predictable sequence imaginable.
Human developers name things after *what they represent*, not *what they are*.

### Universal Replacements (Use When No Domain Context Available)

| AI Generic Name | Human Alternatives (pick contextually) |
|:----------------|:---------------------------------------|
| `data` | `payload`, `blob`, `raw`, `content`, `body`, `chunk` |
| `result` | `outcome`, `out`, `resp`, `matched`, `found`, `rv` |
| `results` | `hits`, `matches`, `rows`, `records`, `entries` |
| `temp` / `tmp` | `scratch`, `buf`, `staging`, `interim` |
| `items` | `entries`, `elements`, `batch`, `records`, `nodes` |
| `item` | `entry`, `elem`, `node`, `record`, `row` |
| `output` | `rendered`, `formatted`, `export`, `dump` |
| `input` | `raw_input`, `source`, `incoming`, `prompt_text` |
| `value` / `val` | `score`, `amount`, `level`, `reading`, `metric` |
| `flag` | `is_ready`, `should_retry`, `needs_auth`, `allow_override` |
| `count` / `num` | `n_items`, `total`, `tally`, `freq` |
| `index` / `i` | `idx`, `pos`, `cursor`, `offset`, `slot` |
| `key` | `field`, `attr`, `prop`, `col`, `tag` |
| `config` | `cfg`, `settings`, `opts`, `prefs` |
| `error` | `err`, `exc`, `fault`, `issue` |
| `message` | `msg`, `note`, `alert`, `payload` |
| `response` | `resp`, `reply`, `ack` |
| `request` | `req`, `rpc`, `call` |
| `callback` | `cb`, `handler`, `hook`, `on_done` |
| `function` | `fn`, `op`, `action`, `handler` |
| `process` | `transform`, `handle`, `run`, `execute`, `dispatch` |
| `calculate` | `compute`, `derive`, `resolve`, `eval_` |
| `get_data` | `fetch_payload`, `pull_records`, `load_entries` |
| `check` | `validate`, `verify`, `assert_`, `ensure` |

### Domain-Specific Naming by Context

**Web/API code:**
`user_data` → `profile`, `account_info`, `auth_ctx`
`send_request` → `dispatch_rpc`, `fire_webhook`, `post_event`
`parse_response` → `unpack_resp`, `deserialize_body`

**Data Processing:**
`process_file` → `ingest_csv`, `parse_feed`, `load_snapshot`
`clean_data` → `scrub_nulls`, `normalize_rows`, `strip_outliers`
`transform` → `reshape`, `pivot`, `flatten_nested`

**ML/Scientific:**
`train_model` → `fit_estimator`, `run_epoch`, `optimize_params`
`predict` → `infer`, `score_batch`, `forward_pass`
`features` → `feature_vec`, `X_train`, `embeddings`

## Naming Rhythm and Inconsistency

Real humans are **inconsistent** in predictable ways:
- They might use `cfg` in one file and `config` in another.
- They abbreviate names they type frequently but spell out rare ones.
- They use longer names for public APIs and shorter ones for local variables.
- They sometimes use `snake_case` for one helper and a slightly different
  convention for another (e.g., `_do_parse` vs `_validate_input`).

**Introduce 1-2 minor naming inconsistencies per file** to break the
uniformity that detectors flag.

## Function Name Patterns

AI functions follow predictable `verb_noun` patterns: `get_user`, `set_value`,
`calculate_total`, `process_data`, `validate_input`.

Human alternatives:
- Omit the verb for getters: `user()` instead of `get_user()`
- Use domain verbs: `enroll()`, `provision()`, `reconcile()`, `hydrate()`
- Use callbacks-style names: `on_connect()`, `after_save()`, `pre_validate()`
- Use boolean-prefixed names: `is_valid()`, `has_access()`, `can_retry()`
- Use underscore prefix for internal: `_resolve_tenant()`, `_build_query()`
