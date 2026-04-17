# SPL Gotchas

## Field & Value Handling

- **Case sensitivity in field values** — `status=OK` won't match `status=ok`. Field *names*
  are case-insensitive; field *values* are not.

- **Quoted vs unquoted strings** — `src=10.0.0.1` and `src="10.0.0.1"` behave the same,
  but multiword values without quotes silently break your search.

- **Wildcards only work as suffix by default** — `host=*web*` works but is expensive.
  Leading wildcards like `*web` force a full scan, killing performance.

- **Null fields in eval** — `eval x=field+1` where field is null produces null, not an error.
  Downstream stats on x will silently drop those events.

- **Multivalue fields in stats** — `stats count by mvfield` splits on each value.
  One event with three values in a field counts as three rows.

- **values() vs list()** — `values()` deduplicates, `list()` preserves duplicates and order.
  Mixing them up gives wrong cardinality.


## Time
- **Default time is the UI picker, not your search** — if you hardcode `earliest` and `latest`
  in SPL but the UI picker overrides them, the UI wins unless you use `index=... earliest=...`
  with "use time range from search" disabled.

- **_time is epoch** — comparing `_time` directly to a human-readable string will silently fail.
  Use `strptime()` or `relative_time()`.

- **timechart span alignment** — `span=1d` aligns to midnight UTC, not your local timezone.
  Results look shifted if your TZ differs.

- **transaction and time** — `transaction` uses wall-clock time between events, not index time.
  Out-of-order ingestion breaks maxspan logic.


## Stats & Aggregation
- **stats wipes non-grouped fields** — any field not in `by` clause or an aggregation
  function is dropped entirely from results.

- **eventstats vs stats** — `eventstats` adds aggregate fields back to every raw event.
  `stats` collapses to one row per group. Confusing them loses your raw events.

- **chart vs timechart** — `chart` needs you to specify the x-axis explicitly.
  `timechart` always uses `_time`. Mixing them up produces unexpected pivots.

- **top and rare default to 10 rows** — results silently truncate. Add `limit=0` for all rows.

- **count vs dc** — `count` counts total events; `dc` counts unique values. Easy to report
  wrong metric without noticing.


## Search Behaviour
- **OR precedence** — `src=a OR src=b AND dst=c` evaluates AND first.
  Always parenthesise OR clauses: `(src=a OR src=b) AND dst=c`.

- **Subsearch result cap** — subsearches return a max of 10,000 results by default.
  Silent truncation causes incomplete joins with no warning.

- **join is not SQL join** — default is inner join. Outer join requires `type=outer`.
  Also limited to 50,000 results by default.

- **append runs sequentially** — it's not a union; the appended search runs after the main
  search completes, doubling your runtime.

- **map iteration limit** — `map` defaults to 10 iterations max. Raises no error, just stops.
  Set `maxsearches=N` explicitly.


## Lookup Gotchas
- **lookup is case-sensitive on match fields** — `host=Webserver` won't match `webserver`
  in your lookup CSV unless you normalise first with `lower()`.

- **outputlookup overwrites silently** — no confirmation, no backup. One bad search
  destroys your lookup table.

- **Automatic lookups fire before your search** — if a field name collides with a lookup
  output field, the lookup value can overwrite your raw event field unexpectedly.



## Performance
- **Transforming commands early kills parallelism** — put `stats`, `eval`, `rex` as late
  as possible. Filter with `search` and `where` first.

- **rex is expensive** — it runs on every event. Extract fields at index time or use
  field extractions in props.conf where possible.

- **NOT and != both scan broadly** — neither can use bloom filters or index optimisation.
  Prefer positive inclusion filters wherever possible.

- **fields + early projection** — use `fields` after filtering to drop unused fields early.
  Reduces memory and speeds up downstream commands.


## Rex & Regex
- **rex named capture groups are required** — `rex "(\d+)"` does nothing useful.
  You need `rex "(?<myfield>\d+)"`.

- **rex default field is _raw** — if you want to regex a specific field:
  `rex field=uri "(?<path>/[^?]+)"`. Forgetting this parses raw event text instead.

- **replace mode in rex** — `rex mode=sed` uses sed syntax, not PCRE. Mixing them breaks
  silently and returns the original string unchanged.

## Misc
- **rename before stats** — renaming a field after stats refers to the new name.
  If you rename after, the stats clause never sees it.

- **fieldformat is cosmetic only** — it changes display, not the underlying value.
  Downstream evals on a fieldformat-modified field see the original value.

- **makeresults is not random** — `makeresults count=5` gives 5 identical events with the
  same `_time`. Use `eval` with `random()` to differentiate them.

- **inputlookup vs lookup** — `inputlookup` reads the whole table as a dataset.
  `lookup` enriches existing results. Using `inputlookup` in a pipeline mid-search
  replaces your results, not enriches them.

- **Acceleration and summary indexes** — report acceleration only kicks in when your
  search exactly matches the accelerated report. One extra field breaks it silently.