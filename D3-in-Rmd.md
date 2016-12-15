# Getting R and D3.js to play nicely in your .Rmd files
@becausealice2  
December 7, 2016  



As there are already plenty of good resources made by people much better qualified to create them, this is not intended to be any kind of comprehensive tutorial on JavaScript, D3.js, R, or R markdown. Instead, its purpose is to show what experience has taught about working with all of them in a single file.

To keep this tutorial simple, let's assume the data has been fully scienced and all that's left is to create the D3 visualization(s). Have a look at raw `cars` sample data set that comes packaged with R


```r
head(cars)
```

```
##   speed dist
## 1     4    2
## 2     4   10
## 3     7    4
## 4     7   22
## 5     8   16
## 6     9   10
```

Passing data between languages is not as simple as referencing the name of the variable that contains them. If you try and add the JavaScript code `console.log(cars);` after the above R chunk, it will return a `ReferenceError` as `cars` has not yet been defined in JavaScript.

You could write the data to a new file that you would then read back in with the appropriate loading method in D3. However, it is possible to pass the data directly to JavaScript inside the .Rmd file.


```r
cat(
  paste(
  '<script>
    var data = ',cars,';
  </script>'
  , sep="")
)
```

```
## <script>
##     var data = c(4, 4, 7, 7, 8, 9, 10, 10, 10, 11, 11, 12, 12, 12, 12, 13, 13, 13, 13, 14, 14, 14, 14, 15, 15, 15, 16, 16, 17, 17, 17, 18, 18, 18, 18, 19, 19, 19, 20, 20, 20, 20, 20, 22, 23, 24, 24, 24, 24, 25);
##   </script> <script>
##     var data = c(2, 10, 4, 22, 16, 10, 18, 26, 34, 17, 28, 14, 20, 24, 28, 26, 34, 34, 46, 26, 36, 60, 80, 20, 26, 54, 32, 40, 32, 40, 50, 42, 56, 76, 84, 36, 46, 68, 32, 48, 52, 56, 64, 66, 54, 70, 92, 93, 120, 85);
##   </script>
```

The above code will, indeed, pass the data into our JavaScript space; however, we have created two new problems for ourselves:

* Each column in the data is being passed as an R vector, which JavaScript will interpret as being a call to some function `c()` with each of the values being passed as parameters.
* The columns were passed one-by-one to the JavaScript `data` variable, overwriting the values in the `speed` column with those in the `dist` column.

To solve both problems at once, take advantage of D3's assumption that the data it works with is in JSON format. R's `jsonlite` library is lightweight with a simple `toJSON()` method that works perfectly for our purpose.


```r
library("jsonlite")
cat(
  paste(
  '<script>
    var data = ',toJSON(cars),';
  </script>'
  , sep="")
)
```

```
## <script>
##     var data = [{"speed":4,"dist":2},{"speed":4,"dist":10},{"speed":7,"dist":4},{"speed":7,"dist":22},{"speed":8,"dist":16},{"speed":9,"dist":10},{"speed":10,"dist":18},{"speed":10,"dist":26},{"speed":10,"dist":34},{"speed":11,"dist":17},{"speed":11,"dist":28},{"speed":12,"dist":14},{"speed":12,"dist":20},{"speed":12,"dist":24},{"speed":12,"dist":28},{"speed":13,"dist":26},{"speed":13,"dist":34},{"speed":13,"dist":34},{"speed":13,"dist":46},{"speed":14,"dist":26},{"speed":14,"dist":36},{"speed":14,"dist":60},{"speed":14,"dist":80},{"speed":15,"dist":20},{"speed":15,"dist":26},{"speed":15,"dist":54},{"speed":16,"dist":32},{"speed":16,"dist":40},{"speed":17,"dist":32},{"speed":17,"dist":40},{"speed":17,"dist":50},{"speed":18,"dist":42},{"speed":18,"dist":56},{"speed":18,"dist":76},{"speed":18,"dist":84},{"speed":19,"dist":36},{"speed":19,"dist":46},{"speed":19,"dist":68},{"speed":20,"dist":32},{"speed":20,"dist":48},{"speed":20,"dist":52},{"speed":20,"dist":56},{"speed":20,"dist":64},{"speed":22,"dist":66},{"speed":23,"dist":54},{"speed":24,"dist":70},{"speed":24,"dist":92},{"speed":24,"dist":93},{"speed":24,"dist":120},{"speed":25,"dist":85}];
##   </script>
```

<script>
    var data = [{"speed":4,"dist":2},{"speed":4,"dist":10},{"speed":7,"dist":4},{"speed":7,"dist":22},{"speed":8,"dist":16},{"speed":9,"dist":10},{"speed":10,"dist":18},{"speed":10,"dist":26},{"speed":10,"dist":34},{"speed":11,"dist":17},{"speed":11,"dist":28},{"speed":12,"dist":14},{"speed":12,"dist":20},{"speed":12,"dist":24},{"speed":12,"dist":28},{"speed":13,"dist":26},{"speed":13,"dist":34},{"speed":13,"dist":34},{"speed":13,"dist":46},{"speed":14,"dist":26},{"speed":14,"dist":36},{"speed":14,"dist":60},{"speed":14,"dist":80},{"speed":15,"dist":20},{"speed":15,"dist":26},{"speed":15,"dist":54},{"speed":16,"dist":32},{"speed":16,"dist":40},{"speed":17,"dist":32},{"speed":17,"dist":40},{"speed":17,"dist":50},{"speed":18,"dist":42},{"speed":18,"dist":56},{"speed":18,"dist":76},{"speed":18,"dist":84},{"speed":19,"dist":36},{"speed":19,"dist":46},{"speed":19,"dist":68},{"speed":20,"dist":32},{"speed":20,"dist":48},{"speed":20,"dist":52},{"speed":20,"dist":56},{"speed":20,"dist":64},{"speed":22,"dist":66},{"speed":23,"dist":54},{"speed":24,"dist":70},{"speed":24,"dist":92},{"speed":24,"dist":93},{"speed":24,"dist":120},{"speed":25,"dist":85}];
  </script>

###### ***\*\*you have to include the results="asis" option in the curly braces at the top of the code chunk to pass the data through\*\****

Now JavaScript has the data, and in a format it can work with. It's time to start D3ing!

There are two ways you can load the D3 library. Include `<script src="https://d3js.org/d3.v4.min.js"></script>` directly in your markdown file at any point before your visualization. Or else create a .html file with only that same script tag in the same directory as your .Rmd file and [include it](http://rmarkdown.rstudio.com/html_document_format.html#includes)

You're now free to visualize with D3.js in R markdown until your hearts content! Just add the code with a JavaScript code chunk, between script tags directly in the markdown, or in a separate file that you link to with a script tag.

<div id="plot"></div>

<script>

// set the dimensions and margins of the graph
var margin = {top: 20, right: 20, bottom: 30, left: 50},
    width = 900 - margin.left - margin.right,
    height = 500 - margin.top - margin.bottom;

// set the ranges
var x = d3.scaleLinear().range([0, width]);
var y = d3.scaleLinear().range([height, 0]);

// append the svg object to the body of the page
// appends a 'group' element to 'svg'
// moves the 'group' element to the top left margin
var svg = d3.select("#plot").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform",
          "translate(" + margin.left + "," + margin.top + ")");


// format the data
data.forEach(function(d) {
    d.speed = +d.speed;
    d.dist  = +d.dist;
});

// Scale the range of the data
x.domain(d3.extent(data, function(d) { return d.speed; }));
y.domain([0, d3.max(data, function(d) { return d.dist; })]);

// Add the scatterplot
svg.selectAll("dot")
    .data(data)
  .enter().append("circle")
    .attr("r", 5)
    .attr("cx", function(d) { return x(d.speed); })
    .attr("cy", function(d) { return y(d.dist); });

// Add the X Axis
svg.append("g")
    .attr("transform", "translate(0," + height + ")")
    .call(d3.axisBottom(x));

// Add the Y Axis
svg.append("g")
    .call(d3.axisLeft(y));

</script>

I should add a word of caution. If you are the type to just copy/paste from <https://bl.ocks.org>, there is no shame--I don't personally know anyone who uses D3 who doesn't do it--but you have to keep in mind their data is always read in from external sources via D3's loading methods. You'll have to delete those lines of code, and substitute in your data variable inside of D3's `.data()` method. Also, be sure to go through the D3 code to update all data references to your column names.
