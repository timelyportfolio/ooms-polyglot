Jeroen Ooms ([@opencpu](https://jeroenooms.github.io/)) provides `R`
users a magical polyglot world of R, JavaScript, C, and C++. This is my
attempt to both thank him and highlight some of all that he has done.
Much of my new R depends on his work.

Ooms' Packages
--------------

[metacran](http://www.r-pkg.org/) provides a list of all [Jeroen's CRAN
packages](http://www.r-pkg.org/maint/jeroen.ooms@stat.ucla.edu). Now, I
wonder if any of his packages are in the *Top Downloads*.

### jsonlite

Let's leverage the helpful meta again from
[metacran](http://www.r-pkg.org/) and very quickly get some assistance
from *hint-hint*
[`jsonlite`](https://cran.r-project.org/web/packages/jsonlite/index.html).

    library(jsonlite)
    library(formattable)
    library(tibble)
    library(magrittr)

    fromJSON("http://cranlogs.r-pkg.org/top/last-month/9") %>%
      {as_tibble(rank=rownames(.$downloads),.$downloads)} %>%
      rownames_to_column(var = "rank") %>%
      format_table(
        formatters = list(
          area(row=which(.$package=="jsonlite")) ~ formatter("span", style="background-color:#D4F; width:100%")
        )
      )

<table class="table table-condensed">
<thead>
<tr>
<th style="text-align:right;">
rank
</th>
<th style="text-align:right;">
package
</th>
<th style="text-align:right;">
downloads
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
Rcpp
</td>
<td style="text-align:right;">
236316
</td>
</tr>
<tr>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
plyr
</td>
<td style="text-align:right;">
208609
</td>
</tr>
<tr>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
ggplot2
</td>
<td style="text-align:right;">
201959
</td>
</tr>
<tr>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
stringi
</td>
<td style="text-align:right;">
188252
</td>
</tr>
<tr>
<td style="text-align:right;">
<span style="background-color:#D4F; width:100%">5 </span>
</td>
<td style="text-align:right;">
<span style="background-color:#D4F; width:100%">jsonlite</span>
</td>
<td style="text-align:right;">
<span style="background-color:#D4F; width:100%">175853 </span>
</td>
</tr>
<tr>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
digest
</td>
<td style="text-align:right;">
174714
</td>
</tr>
<tr>
<td style="text-align:right;">
7
</td>
<td style="text-align:right;">
stringr
</td>
<td style="text-align:right;">
173835
</td>
</tr>
<tr>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
magrittr
</td>
<td style="text-align:right;">
166437
</td>
</tr>
<tr>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
scales
</td>
<td style="text-align:right;">
156694
</td>
</tr>
</tbody>
</table>
`jsonlite` is an ultra-fast reliable tool to convert and create `json`
in `R`. It's fast because like much Jeroen's work, he leverages
`C`/`C++` libraries. `shiny` and `htmlwidgets` both depend on
`jsonlite`.

### V8

[`V8`](https://github.com/jeroenooms/V8) gives `R` its own embedded
JavaScript engine to leverage functionality in JavaScript that might not
exist in `R`. For example, the
[`WebCola`](http://marvl.infotech.monash.edu/webcola) constraint-based
layout engine offers valuable technology not available within R. Let's
partially recreate the [smallgroups
example](http://marvl.infotech.monash.edu/webcola/examples/smallgroups.html)
all in R. You might notice that the previously mentioned `jsonlite` is
essential to this workflow.

    library(V8)
    library(jsonlite)
    library(scales)

    ctx = new_context(global="window")

    ctx$source("https://cdn.rawgit.com/tgdwyer/WebCola/master/WebCola/cola.min.js")

    ## [1] "true"

    ### small grouped example
    group_json <- fromJSON(
      system.file(
        "htmlwidgets/lib/WebCola/examples/graphdata/smallgrouped.json",
        package = "colaR"
      )
    )

    # need to get forEach polyfill
    ctx$source(
      "https://cdnjs.cloudflare.com/ajax/libs/es5-shim/4.1.10/es5-shim.min.js"
    )

    # code to recreate small group example
    js_group <- '
    // console.assert does not exists
    console = {}
    console.assert = function(){};

    var width = 960,
      height = 500

    graph = {
    "nodes":[
      {"name":"a","width":60,"height":40},
      {"name":"b","width":60,"height":40},
      {"name":"c","width":60,"height":40},
      {"name":"d","width":60,"height":40},
      {"name":"e","width":60,"height":40},
      {"name":"f","width":60,"height":40},
      {"name":"g","width":60,"height":40}
    ],
    "links":[
      {"source":1,"target":2},
      {"source":2,"target":3},
      {"source":3,"target":4},
      {"source":0,"target":1},
      {"source":2,"target":0},
      {"source":3,"target":5},
      {"source":0,"target":5}
    ],
    "groups":[
      {"leaves":[0], "groups":[1]},
      {"leaves":[1,2]},
      {"leaves":[3,4]}
      ]
    }

    var g_cola = new cola.Layout()
      .linkDistance(100)
      .avoidOverlaps(true)
      .handleDisconnected(false)
      .size([width, height]);

    g_cola
      .nodes(graph.nodes)
      .links(graph.links)
      .groups(graph.groups)
      .start()
    '

    # run the small group JS code in V8
    ctx$eval(js_group)

    ## [1] "[object Object]"

Now, `WebCola` has done the hard work and laid out our nodes and links,
so let's get their positions.

    nodes <- ctx$get('
      graph.nodes.map(function(d){
        return {name: d.name, x: d.x, y: d.y, height: d.height, width: d.width};
      })
    ')

    links <- ctx$get('
      graph.links.map(function(d){
        return {x1: d.source.x, y1: d.source.y, x2: d.target.x, y2: d.target.y}
      })
    ')

Some great examples of packages employing `V8` are
[`geojsonio`](http://www.r-pkg.org/pkg/geojsonio),
[`lawn`](https://github.com/ropensci/lawn),
[`DiagrammeRsvg`](http://www.r-pkg.org/pkg/DiagrammeRsvg),
[`rmapshaper`](https://github.com/ateucher/rmapshaper), and
[`daff`](https://github.com/edwindj/daff).

### rjade

We got layout coordinates above. Let's use another one of Jeroen's
packages [`rjade`](https://github.com/jeroenooms/rjade) that provides
[`jade`](http://jade-lang.com/) (now called
[pug](https://github.com/pugjs/pug)) templates through `V8`. `rjade`
will let us build a `SVG` graph with our layout.

    library(rjade)
    library(htmltools)

    svg <- jade_compile(
    '
    doctype xml
    svg(version="1.1",xmlns="http://www.w3.org/2000/svg",xmlns:xlink="http://www.w3.org/1999/xlink",width="960px",height="500px")
      each l in lines
        line(style={fill:none, stroke:"lightgray"})&attributes({"x1": l.x1, "x2": l.x2, "y1": l.y1, "y2": l.y2})
      each val in rects
        g
          rect(style={fill: fillColor})&attributes({"x": val.x - val.width/2, "y": val.y - val.height/2, "height": val.height - 6, "width": val.width - 6, rx: 5, ry: 5})
          text&attributes({"x": val.x, "y": val.y, "dy": ".2em", "text-anchor":"middle"})= val.name
    '
          ,pretty=T
    )(rects = nodes, lines = links, fillColor = "lightgray")

    HTML(svg)

<!--html_preserve-->
<?xml version="1.0" encoding="utf-8" ?>
<svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="960px" height="500px">
<line style="fill:undefined;stroke:lightgray" x1="390.24" x2="417.6291" y1="333.6926" y2="240.0374"></line>
<line style="fill:undefined;stroke:lightgray" x1="417.6291" x2="495.0286" y1="240.0374" y2="161.6408"></line>
<line style="fill:undefined;stroke:lightgray" x1="495.0286" x2="495.0286" y1="161.6408" y2="58.0727"></line>
<line style="fill:undefined;stroke:lightgray" x1="488.7193" x2="390.24" y1="322.5478" y2="333.6926"></line>
<line style="fill:undefined;stroke:lightgray" x1="417.6291" x2="488.7193" y1="240.0374" y2="322.5478"></line>
<line style="fill:undefined;stroke:lightgray" x1="495.0286" x2="566.5713" y1="161.6408" y2="244.9418"></line>
<line style="fill:undefined;stroke:lightgray" x1="488.7193" x2="566.5713" y1="322.5478" y2="244.9418"></line>
<g>
<rect style="fill:lightgray" x="458.7193" y="302.5478" height="34" width="54" rx="5" ry="5"></rect>
<text x="488.7193" y="322.5478" dy=".2em" text-anchor="middle">a</text>
</g> <g>
<rect style="fill:lightgray" x="360.24" y="313.6926" height="34" width="54" rx="5" ry="5"></rect>
<text x="390.24" y="333.6926" dy=".2em" text-anchor="middle">b</text>
</g> <g>
<rect style="fill:lightgray" x="387.6291" y="220.0374" height="34" width="54" rx="5" ry="5"></rect>
<text x="417.6291" y="240.0374" dy=".2em" text-anchor="middle">c</text>
</g> <g>
<rect style="fill:lightgray" x="465.0286" y="141.6408" height="34" width="54" rx="5" ry="5"></rect>
<text x="495.0286" y="161.6408" dy=".2em" text-anchor="middle">d</text>
</g> <g>
<rect style="fill:lightgray" x="465.0286" y="38.0727" height="34" width="54" rx="5" ry="5"></rect>
<text x="495.0286" y="58.0727" dy=".2em" text-anchor="middle">e</text>
</g> <g>
<rect style="fill:lightgray" x="536.5713" y="224.9418" height="34" width="54" rx="5" ry="5"></rect>
<text x="566.5713" y="244.9418" dy=".2em" text-anchor="middle">f</text>
</g> <g>
<rect style="fill:lightgray" x="449.9339" y="355.6926" height="34" width="54" rx="5" ry="5"></rect>
<text x="479.9339" y="375.6926" dy=".2em" text-anchor="middle">g</text>
</g>
</svg>
<!--/html_preserve-->
### rsvg

If we are not in the browser though with inline `SVG` support, we very
likely will want a static image format such as `png` or `jpeg`. Of
course, Jeroen has that covered also with the crazy-speedy
[`rsvg`](https://github.com/jeroenooms/rsvg). Jeroen offers
[`base64`](https://github.com/jeroenooms/base64), but in this case we
will use `base64enc`, since it allows `raw`.

    library(rsvg)
    library(base64enc)

    graph_png <- rsvg_png(charToRaw(svg))

    tags$img(src=dataURI(graph_png), mime="image/png")

<!--html_preserve-->
<img src="data:;base64,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" mime="image/png"/><!--/html_preserve-->

### magick

Jeroen's newest package [`magick`](https://github.com/jeroenooms/magick)
is in my mind the coolest. `magick` gives us all the power of
[`ImageMagick`](http://www.imagemagick.org/script/index.php) as easy `R`
functions, and is pure wizardry. I am still shocked that it compiled
first try with absolutely no problems.

    library(magick)

    graph_img <- image_read(graph_png)
    wizard_img <- image_read("http://www.imagemagick.org/image/wizard.png")

    images <- image_annotate(
      image_append(
        c(
          image_scale(image_crop(wizard_img, "600x600+100+100"), "100"),
          image_crop(graph_img, "400x400+200+0")
        )
      ),
      "Ooms is a Wizard!",
      size = 20,
      color = "blue",
      location = "+100+200"
    )

    tags$img(src=dataURI(image_write(images)), mime="image/png")

<!--html_preserve-->
<img src="data:;base64,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" mime="image/png"/><!--/html_preserve-->

### commonmark

I should note that this document was assembled in `rmarkdown`. RStudio
gives us lots of tools for working with `rmarkdown`, but Jeroen gives us
a powerful tool
[`commonmark`](https://github.com/jeroenooms/commonmark). Let's use it
to give our readers other options for output.

    library(commonmark)

    rmarkdown::render("Readme.Rmd", "Readme.md", output_format="md_document")

    tex <- markdown_latex(readLines("Readme.md"))
    cat(tex, file="Readme.tex")

<!--html_preserve-->
<a href="./Readme.tex" alt="link to tex version">rendered as
tex</a><!--/html_preserve-->

Conclusion and Thanks
---------------------

There are of course more packages, but I'll stop here. Jeroen Ooms truly
is a wizard, and the `R` community is extraordinarily blessed to have
him. Thanks so much Jeroen.

For even more wizardry, be sure to check out
[opencpu](https://www.opencpu.org/apps.html) from Jeroen, which makes R
available as a web service.
