### csv to hierarchy with d3 in R V8

<img src = "https://timelyportfolio.github.io/ooms-polyglot/d3_in_R_v8_console.gif"></img>


```
library(V8)

ctx <- v8()

ctx$source("https://d3js.org/d3-hierarchy.v1.min.js")
# use jsan to handle cyclic json
ctx$source("https://cdn.rawgit.com/timelyportfolio/jsan/master/jsan.js")


# use d3.stratify example from d3-hierarchy
#  https://github.com/d3/d3-hierarchy#stratify
hier <- read.csv(textConnection(
'name,parent
Eve,
Cain,Eve
Seth,Eve
Enos,Seth
Noam,Seth
Abel,Eve
Awan,Eve
Enoch,Awan
Azura,Eve
'
), stringsAsFactors = FALSE)

ctx$assign("table", hier)

# for copy/paste in console
#  note: needs to be on one line
#var root = d3.stratify().id(function(d) { return d.name; }).parentId(function(d) { return d.parent; })(table);
ctx$console()

jsonlite::fromJSON(
  ctx$get('jsan.stringify(root)'),
  simplifyDataFrame = FALSE
)
```
