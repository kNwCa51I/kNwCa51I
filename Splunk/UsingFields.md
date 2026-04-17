# Uring Fields
Sections
- Using Fields
- Using The Sidebar
- Fields in search Queries
- Fields in search Results
- Enriching Data with Knowledge Objects
- Additional Resources

Learn how to use fields ot filter data to get the results we need. 
Learn how to use feilds 
= feilds Extracted at index vs search time 
- Difference in Persistent and Temporary Fields: eg
    (`action`,`bandwidth`,`bytes`,`catagoryId`,`clientip`)
how ot do more with `KO`'s

## Using The Sidebar
Will display `Selected` and `Interesting` 
- Interesting will be prefaces with with either a ...:
  - `a` for a string value
  - `#` for a numeric value

Some fields are automatically extracted from data based on the source type but additional fields can be manually extracted (`action` ,`catagoryID` ,`clientip` ,`product_name`) but additional fields can be extracted `Field Extractions` for additional insight. `Calculated Fields` are added to events at search times based on the values of existing fields.


## Fields in search Queries
- Field `names` are case sensitive
- Field `values` are not case sensitive

### Field Operators
Numerical operators
- `=` or `!=`
String operators
- `>` , `>=` 
- `<` , `<=`
- wild cards and quotes can be used too `("This*" OR That*)`
Logical/Boolean operators in order of Evaluation :
- `NOT` -> `OR` -> `AND` 

## !! Caution between  `!=` and `NOT`
**`status!=200`** — inequality operator
- Matches events where `status` **exists** and is not 200
- Events where `status` field is **absent or null** are **excluded**

**`NOT status=200`** — negation of a positive match
- Matches events where `status` is not 200
- Events where `status` field is **absent or null** are **included**

### `IN` operator 
The is more elegant:

```
index=myindex status IN ("500", "404", "200")
```
...than 

```
index=myindex (status=500 OR status=404 OR status=200)
```
**Misc** 
Searching for ip address wiht wildcards:
```
clientip=123.50.*
```

**Fields Command**
Can be used ot *include* or *exclude* fields for example 
```
| fields status
```
Putting this before the stats ir more elegant 

**rename command** 
- Separate multiple fields with a comma
- new labels as phases need ot be wrapped in `"` 
eg;
```
| status as "HTTP Stats", count as "Numebr of Events"
```

## Fields in search Results

### Index Time vs Search Time

#### What does it mean?

When Splunk ingests raw data, it has two opportunities to extract and enrich fields:

- **Index time** — happens once, as the event is written to disk
- **Search time** — happens on demand, every time a search runs

---

#### Index Time

Splunk processes the raw event and permanently extracts certain fields before storing it.

Fields extracted at index time:
- `_time` — timestamp
- `host` — source host
- `source` — file or input path
- `sourcetype` — data classification
- `index` — which index it lands in
- Any custom fields defined in `transforms.conf` with `INDEXED_EXTRACTIONS`

**Pros:** Extremely fast to query — fields are pre-built, stored, and indexed.  
**Cons:** Rigid. You cannot change them without re-indexing the data. Increases storage cost.

> Rule of thumb: only extract at index time if you *know* you will always need that field
> and query performance is critical (e.g. high-volume security events).

---

#### Search Time

The majority of field extraction in Splunk happens here. When you run a search,
Splunk reads the raw `_raw` event text and extracts fields on the fly.

Mechanisms that fire at search time:
- **Field extractions** — regex or delimiter rules defined in `props.conf` / `transforms.conf`
- **Lookups** — automatic or manual enrichment from lookup tables
- **`Calculated Fields`** — fields derived from other fields via `eval` in `props.conf`
- **Field aliases** — alternate names mapped to existing fields
- **eval in SPL** — ad hoc field creation during a search

**Pros:** Flexible. Add new fields without touching stored data. Schema-on-read.  
**Cons:** CPU cost paid on every search. High-cardinality extractions over large datasets
can be slow.

---

#### Temporary Fields with `eval`

`eval` creates or modifies fields that exist only for the duration of that search.
They are never written to disk.

```spl
index=web | eval response_kb = bytes / 1024
```

The field `response_kb` exists in your results but not in the underlying events.

**Can you run code with eval?**  
Not arbitrary code — but `eval` is a surprisingly capable expression engine. You can:
- Arithmetic:          `eval total = price * quantity`
- String manipulation: `eval upper_host = upper(host)`
- Conditionals:        `eval severity = if(status>=500, "critical", "ok")`
- Type conversion:     `eval epoch = strptime(timestamp, "%Y-%m-%d")`
- Regex replacement:   `eval clean = replace(uri, "\?.*", "")`
- Chained logic:       `eval tier = case(score>90,"gold", score>50,"silver", true(),"bronze")`

It is a functional expression language, not a scripting runtime. You cannot write loops,
define functions, import libraries, or execute shell commands from within `eval`.

For heavier logic, you would reach for:
- **Custom search commands** — Python scripts registered as SPL commands
- **Scripted inputs** — Python/shell running at collection time
- **Splunk SOAR** — for orchestration and code execution in response workflows

---

#### Summary Table

| Property             | Index Time                  | Search Time                  |
|----------------------|-----------------------------|------------------------------|
| When it runs         | Once, on ingest             | Every search execution       |
| Speed                | Fast to query               | Slower on large datasets     |
| Flexibility          | Low — fixed on disk         | High — change anytime        |
| Storage cost         | Higher                      | No additional cost           |
| Where configured     | `transforms.conf`           | `props.conf` / SPL           |
| Example fields       | `host`, `source`, `_time`   | `eval`, lookups, regex       |

```
index=network sourcetype=cisco_wsa_squid
| stats sum(sc_byts) as Bytes by usage
```
To get megabytes

```
index=network sourcetype=cisco_wsa_squid
| stats sum(sc_byts) as Bytes by usage
| eval bandwidth = Bytes/1024/1024
```


## erex and rex commands

# erex and rex Commands

---

## erex (Easy Regex)

Automatically infers a regex pattern from examples you provide, rather than writing
one manually. Splunk inspects the examples, finds common surrounding context in the
raw events, and generates the extraction for you.

### Syntax

```spl
| erex <fieldname> fromfield=<source_field> examples="<val1>, <val2>" [counterexamples="<val3>"]
```

### Example

```spl
index=games sourcetype=SimCubeBeta
| erex Character fromfield=_raw examples="pixie, Kooby"
| where isnull(Character)
```

**What's happening here:**
- `erex` looks at `_raw` events containing "pixie" and "Kooby"
- Infers a regex pattern that would match both
- Creates a new field called `Character` using that pattern
- `where isnull(Character)` surfaces events where the pattern *failed* to match — useful for QA-ing whether the inferred regex is catching everything it should

### Inspecting the Generated Regex

Go to **Job > Inspect Job** (or the Jobs menu) after running — Splunk shows you the regex it inferred. Always do this. Copy it out, validate it on `regex101.com`, then migrate it to a proper `rex` command or a field extraction in `props.conf`.

### counterexamples

Tell erex what *not* to match, tightening the inferred pattern:

```spl
| erex Character fromfield=_raw examples="pixie, Kooby" counterexamples="ERROR, NULL"
```

### Gotchas

- Spelling of field names is not validated — typos silently create a null field
- Works best with consistent surrounding structure (delimiters, fixed prefixes)
- Not suitable for production — treat it as a pattern discovery tool only
- The more varied your examples, the more general (and potentially noisy) the regex

---

## rex

Extracts fields from events using explicit regex patterns you write yourself.
More powerful and precise than `erex`, but requires you to know your regex.
Uses **PCRE** syntax with **named capture groups** to define new field names.

### Syntax

```spl
| rex [field=<source_field>] "<regex_with_named_groups>"
| rex mode=sed "<sed_expression>"
```

### Basic Extraction Example

```spl
index=web
| rex field=_raw "user=(?<Username>[a-zA-Z0-9_.]+)"
```

- Creates a new field `Username` from whatever matches inside the capture group
- `field=_raw` is the default — omitting it parses `_raw` anyway

### Extracting from a Specific Field

```spl
index=web
| rex field=uri "(?<Path>/[^?]+)\??(?<QueryString>.*)"
```

- Always specify `field=` when your target isn't `_raw`
- Forgetting this parses raw event text and gives confusing results

### Multiple Named Groups in One rex

```spl
| rex "src=(?<SrcIP>\d+\.\d+\.\d+\.\d+)\s+dst=(?<DstIP>\d+\.\d+\.\d+\.\d+)"
```

One `rex` call can extract multiple fields simultaneously — no need to chain multiple calls for patterns that appear in the same region of the event.

### Chaining rex Commands

```spl
index=auth
| rex "user=(?<Username>[a-zA-Z0-9_.]+)"
| rex field=Username "(?<Domain>[^@]+)$"
```

Chain rex calls to extract from previously extracted fields.

### sed Mode (Find and Replace)

```spl
| rex mode=sed "s/password=[^ ]*/password=REDACTED/g"
```

- Uses **sed syntax**, not PCRE — these are not interchangeable
- Useful for masking sensitive values before further processing or output
- Modifies the field in place (default is `_raw`)
- Specify `field=` to target another field: `| rex field=uri mode=sed "s/token=[^&]*/token=REDACTED/g"`

### Performance

`rex` runs on every event in the search results — it is applied post-retrieval, not at index time. This makes it expensive on large datasets.

**Mitigation strategies:**
- Filter aggressively with `search` and `where` *before* `rex` to reduce the event count it operates on
- For patterns you use repeatedly, move them to `props.conf` as indexed field extractions — they run once at ingest time
- Use `fields` to drop unused fields early in the pipeline before `rex` runs

### Gotchas

| Gotcha | Detail |
|--------|--------|
| Named groups required | `rex "(\d+)"` extracts nothing — must use `(?<fieldname>\d+)` |
| Python syntax won't work | `(?P<fieldname>...)` is Python; Splunk requires `(?<fieldname>...)` |
| sed vs PCRE | `mode=sed` uses sed syntax — mixing the two breaks silently, returning the original string unchanged |
| Default field is `_raw` | Always specify `field=` when targeting anything else |
| Greedy matching | Quantifiers like `.*` are greedy by default — use `.*?` for non-greedy where needed |
| No match = null | If the pattern doesn't match, the field is null — not an empty string, not an error |

### erex vs rex — When to Use Which

| Scenario | Use |
|----------|-----|
| Exploring an unfamiliar log format | `erex` — let Splunk infer the pattern first |
| Production pipelines | `rex` — explicit, predictable, documented |
| Repeated extractions across many searches | `props.conf` field extraction — runs at index time |
| Masking or modifying field values | `rex mode=sed` |
| Validating what erex inferred | Jobs menu → copy pattern → refactor into `rex` |


##### Regex fun and games
**Most fun / game-like**
- **Regex Machina** — `codepip.com/games/regex-machina` — classify travellers as human or bot using regex. Actually feels like a game, good pacing, covers all the core concepts.
- **Slash\Escape** — `therobinlord.com/projects/slash-escape` — slasher-horror themed text adventure where you write regex to progress the story. Free, browser-based, genuinely weird and fun.
- **Copy Editor** — on Steam, paid — puzzle game where you use regex find-and-replace to edit famous texts for absurd publishers. Most polished of the lot.
**Puzzle / brain-teaser style**
- **Regex Crossword** — `regexcrossword.com` — crossword where the clues are regex patterns. Surprisingly addictive, free, runs in browser. Good for when you want something low-pressure.
**More structured but still interactive**
- **RegexOne** — `regexone.com` — not a game exactly but short interactive lessons with real practice problems including log parsing and email matching. Closest to what you'd actually use in Splunk.



Splunk intro to Regex
- https://www.youtube.com/watch?v=QvxLJ9f6AK0
---


# Knowledge Objects

### Calcualted Fields


We previopusly used the eval command ot create a temporaty filed at search time 

```
| eval bandwidth = Bytes/1024/1024
```

If we want this field to persist , we ca nstore this int oa `callcualted field`, spliunk wuill make the feild at search time , everytime a search with our `Vytres` field is run.

Ther is one thing to kee pin mind with creating a calcualted field

- `Calculated Fields` can only refrecne fields that are already in the events returned by a search! 
We need to make sure `Calculated Fields` are referencing fields which have already been extracted.


## Field aliases 

Allow you at assign a;lternate names to filed from our data 
eg user  and user name , we can create an alias all grouped for Employee

Then allows All values to be searched at once. Alias do not remove the original field , so we can search for htoriginal name or the alis


### lookups 

allow us to add other fileds to the events which are not part of the index data

Note: I dont uite get thsesew 


## search tiem opperation order when working with objects related ot fields

```
Field Extractions      # Evaluated first in the pipeine
↓
Field Aliases
↓
Calcualted Fields
↓
Look ups 
↓
Event Types
↓
Tags
```

Therefore `calcualted fields` can add additional context to extracted fields but not the other way round. 

