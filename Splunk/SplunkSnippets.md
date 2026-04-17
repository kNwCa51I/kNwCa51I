


# Splunk SPL Functions

## Statistical / Aggregation
- `avg()`          - Returns the average of a numeric field
- `count()`        - Counts the number of events or values
- `dc()`           - Distinct count of unique values in a field
- `earliest()`     - Returns the earliest value by time
- `estdc()`        - Estimates distinct count using probabilistic method
- `exactperc()`    - Returns exact percentile value of a field
- `latest()`       - Returns the most recent value by time
- `list()`         - Returns all values of a field as list
- `max()`          - Returns the maximum value of a field
- `median()`       - Returns the median value of a numeric field
- `min()`          - Returns the minimum value of a field
- `mode()`         - Returns the most frequent value in a field
- `perc()`         - Returns approximate percentile of a field
- `range()`        - Returns difference between max and min values
- `stdev()`        - Returns standard deviation of a numeric field
- `stdevp()`       - Returns population standard deviation of a field
- `sum()`          - Returns the sum of a numeric field
- `sumsq()`        - Returns sum of squares of a numeric field
- `values()`       - Returns all distinct values of a field
- `var()`          - Returns sample variance of a numeric field
- `varp()`         - Returns population variance of a numeric field

## Evaluation / Math
- `abs()`          - Returns absolute value of a number
- `ceiling()`      - Rounds number up to nearest integer
- `exact()`        - Forces exact decimal arithmetic evaluation
- `exp()`          - Returns e raised to the power given
- `floor()`        - Rounds number down to nearest integer
- `ln()`           - Returns natural logarithm of a number
- `log()`          - Returns base-10 logarithm of a number
- `pi()`           - Returns the value of pi
- `pow()`          - Raises a number to specified power
- `round()`        - Rounds number to specified decimal places
- `sigfig()`       - Rounds to significant figures
- `sqrt()`         - Returns square root of a number

## String
- `len()`          - Returns the character length of a string
- `lower()`        - Converts string to lowercase
- `ltrim()`        - Removes leading whitespace or characters
- `replace()`      - Replaces matches in a string with another
- `rtrim()`        - Removes trailing whitespace or characters
- `split()`        - Splits a string on a delimiter into multivalue
- `strcat()`       - Concatenates multiple strings into one
- `substr()`       - Extracts a substring by position and length
- `trim()`         - Removes leading and trailing whitespace
- `upper()`        - Converts string to uppercase
- `urldecode()`    - Decodes a URL-encoded string

## Conversion / Type
- `tostring()`     - Converts a number or value to string
- `tonumber()`     - Converts a string to a number
- `typeof()`       - Returns the data type of a value

## Conditional / Logical
- `case()`         - Returns value based on first true condition
- `cidrmatch()`    - Checks if IP is within a CIDR range
- `coalesce()`     - Returns first non-null value from arguments
- `if()`           - Returns one of two values based on condition
- `in()`           - Checks if a value exists in a list
- `like()`         - Pattern matches a string using wildcards
- `match()`        - Tests if a string matches a regex pattern
- `null()`         - Returns null value
- `nullif()`       - Returns null if two values are equal
- `searchmatch()`  - Tests if a string matches a search expression
- `validate()`     - Returns first value where condition is false

## Time / Date
- `now()`          - Returns current Unix epoch timestamp
- `relative_time()-` Calculates a time relative to another time
- `strftime()`     - Formats epoch time as a human-readable string
- `strptime()`     - Parses a time string into epoch timestamp
- `time()`         - Returns current time as epoch (alias for now)

## Multivalue
- `mvappend()`     - Appends values into a multivalue field
- `mvcount()`      - Returns count of values in a multivalue field
- `mvdedup()`      - Removes duplicate values from a multivalue field
- `mvfilter()`     - Filters multivalue field values by condition
- `mvfind()`       - Finds index of a value in multivalue field
- `mvindex()`      - Returns value at a specific multivalue index
- `mvjoin()`       - Joins multivalue field values with a delimiter
- `mvmap()`        - Applies an expression to each multivalue element
- `mvrange()`      - Creates a multivalue field from a range
- `mvsort()`       - Sorts values in a multivalue field
- `mvzip()`        - Combines two multivalue fields element by element

## Cryptographic / Hashing
- `md5()`          - Returns MD5 hash of a string
- `sha1()`         - Returns SHA-1 hash of a string
- `sha256()`       - Returns SHA-256 hash of a string
- `sha512()`       - Returns SHA-512 hash of a string

## IP / Network
- `cidrmatch()`    - Returns true if IP matches CIDR range

## JSON
- `json_valid()`   - Returns true if string is valid JSON
- `spath()`        - Extracts values from JSON or XML paths

# Splunk SPL Commands

## Search & Filtering
- `search`        - Filters events matching a search expression
- `where`         - Filters results using eval expressions
- `regex`         - Filters events matching a regex pattern
- `head`          - Returns first N results
- `tail`          - Returns last N results
- `dedup`         - Removes duplicate events by field values
- `uniq`          - Removes consecutive duplicate results

## Transformation & Aggregation
- `stats`         - Calculates statistics aggregated over results
- `eventstats`    - Adds aggregate statistics to each event
- `streamstats`   - Running statistics calculated across event stream
- `chart`         - Creates a data table for charting purposes
- `timechart`     - Statistics aggregated over time for charting
- `top`           - Returns most common values of a field
- `rare`          - Returns least common values of a field
- `geostats`      - Aggregates statistics for geographic visualisation

## Field Manipulation
- `eval`          - Creates or modifies fields using expressions
- `rename`        - Renames one or more fields
- `fields`        - Keeps or removes specified fields from results
- `fieldformat`   - Formats field values at display time only
- `convert`       - Converts field values to numeric or time
- `extract`       - Extracts fields using regex from raw events
- `rex`           - Extracts or replaces fields using regex
- `spath`         - Extracts fields from JSON or XML data
- `kvform`        - Extracts key-value pairs from form data
- `erex`          - Infers regex from example field values

## Enrichment & Lookup
- `lookup`        - Adds fields from an external lookup table
- `inputlookup`   - Reads results directly from a lookup table
- `outputlookup`  - Writes results to a lookup table file
- `localop`       - Forces commands to run locally not remote
- `typelearner`   - Suggests event types from search results
- `typer`         - Assigns event types to matching events

## Joins & Combining
- `join`          - Combines results from two datasets by field
- `append`        - Appends subsearch results to main results
- `appendcols`    - Appends subsearch fields as new columns
- `union`         - Merges results from multiple datasets together
- `selfjoin`      - Joins results with themselves on a field
- `appendpipe`    - Appends results of a pipeline to current

## Sorting & Ordering
- `sort`          - Sorts results by one or more fields
- `reverse`       - Reverses the order of results

## Grouping & Transactions
- `transaction`   - Groups related events into single transaction
- `cluster`       - Groups similar events using string similarity
- `concurrency`   - Calculates concurrent events at any time

## Subsearch & Flow
- `map`           - Runs a search for each result iteratively
- `sendemail`     - Sends email with search results attached
- `collect`       - Writes results into a summary index
- `overlap`       - Finds events shared between two datasets

## Formatting & Output
- `table`         - Displays results in column-formatted table
- `transpose`     - Swaps rows and columns in results table
- `untable`       - Converts columnar data back to row format
- `xyseries`      - Converts results into x-y charting format
- `format`        - Formats multivalue results into a string
- `foreach`       - Applies eval expression across matching fields

## Time
- `timewrap`      - Compares timechart data across time periods
- `gentimes`      - Generates time range results between dates

## Machine Learning / Anomaly
- `anomalydetection` - Identifies anomalous field value combinations
- `anomalousvalue`   - Finds statistically unusual field values
- `predict`          - Forecasts future values using historical data
- `kmeans`           - Clusters events using k-means algorithm
- `birch`            - Clusters events using BIRCH algorithm
- `densityfunction`  - Estimates probability density of field values

## Splunk Infrastructure
- `metadata`      - Returns metadata about indexes sources sourcetypes
- `dbinspect`     - Returns information about index bucket contents
- `audit`         - Returns audit trail events from audit index
- `typeahead`     - Returns typeahead suggestions for search terms
- `summarize`     - Manages report acceleration summary data
- `inputcsv`      - Reads results from a CSV file as input
- `outputcsv`     - Writes results to a CSV file
- `makeresults`   - Creates a specified number of empty results
- `mcollect`      - Writes metric data to a metrics index
- `mstats`        - Queries data stored in a metrics index