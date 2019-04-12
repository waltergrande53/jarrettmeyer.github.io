---
title:      Chord Diagrams in D3
layout:     post
date:       2019-04-12
tags:       d3js
---

[Chord diagrams](https://en.wikipedia.org/wiki/Chord_diagram) are a graphical method of displaying relationships within a square matrix. Usually, the matrix shows a transfer from one state to another state.

### Chord Chart Output

<svg id="canvas"></svg>

### Data Table

<table id="table"></table>

*This data is randomly generated by the script.*

In our example data set, <span id="state-1-3"></span> items started in State 1 and went to State 3.

Along the diagonal, we see values that stayed in a single state. In our example data set, <span id="state-2-2"></span> items started in State 2 and remained in State 2.

### The Code

First, our data matrix must be chord-ified. Fortunately, D3 makes this quite easy with the `d3.chord` function.

```js
// Chord-ify the data set.
let chord = d3.chord().padAngle(deg2rad(1));
let chords = chord(data);    
```

Let's define our `svg` object.

```js
let svg = d3.select("#canvas");
svg.attr("width", width)
    .attr("height", height)
    .attr("font-size", fontSize)
    .attr("font-family", fontFamily);
```

The `d3.chord`, `d3.arc`, and `d3.ribbon` functions will create graphical elements centered at the point (0, 0). We need to apply a transform so graphic will appear in the middle of our SVG.

```js
// Define the view window for the chort chart.
let gView = svg.append("g")
    .classed("view", true)
    .attr("transform", `translate(${width / 2}, ${height / 2})`);
```

We have two primary groups to concern ourself with. We have arcs and we have ribbons.

```js
// Create a wrapping group for the chord groups (arcs).
let gGroups = gView.selectAll("g.group")
    .data(chords.groups)
    .join("g")
    .classed("group", true);

// Create a wrapping group for the chord groups.
let gChords = gView.selectAll("g.chord")
    .data(chords)
    .join("g")
    .classed("chord", true);
```

We use our arc generator function to draw the arcs.

```js
// Generator function for the outer arcs.
let arc = d3.arc()
    .innerRadius(innerRadius)
    .outerRadius(outerRadius);

// Create a path, using the arc generator function, for each
// group in the data set.
gGroups.append("path")
    .attr("fill", d => color(d.index, numCategories))
    .attr("stroke", d => d3.rgb(color(d.index, numCategories)).darker())
    .attr("stroke-width", 1)
    .attr("d", arc)
    .append("title")
    .text(d => label(d.index));
```

Let's add some tick marks to the arcs.

```js
/**
 * Function to generate the tick marks for a single arc.
 *
 * @param {object} data
 * @param {number} step
 */
function ticks(data, step) {
    let k = (data.endAngle - data.startAngle) / data.value;
    return d3.range(0, data.value, step).map((x) => {
        return {
            value: x,
            angle: x * k + data.startAngle
        };
    });
}

// Create a group for each small tick mark.
let gTicks = gGroups.selectAll("g.tick")
    .data(d => ticks(d, smallTick))
    .join("g")
    .classed("tick", true)
    .attr("transform", d => `rotate(${rad2deg(d.angle) - 90}) translate(${outerRadius}, 0)`);

// Create a tick for each tick mark.
gTicks.append("line")
    .attr("x1", 0)
    .attr("x2", tickSize)
    .attr("stroke", "black")
    .attr("stroke-width", 1);

// Create a text element for large tick mark.
gTicks.append("text")
    .filter(d => d.value % largeTick === 0)
    .attr("x", tickSize + 2)
    .attr("dy", "0.35em")
    .attr("transform", d => d.angle < Math.PI ? "rotate(0) translate(0)": "rotate(180) translate(-16, 0)")
    .attr("text-anchor", d=> d.angle < Math.PI ? "start" : "end")
    .text(d => format(d.value));
```

Finally, let's draw the ribbons.

```js
// Generator function for the inner chords.
let ribbon = d3.ribbon().radius(innerRadius);

// Create a path, using the ribbon generator function, for each
// path in the data set.
gChords.append("path")
    .attr("d", ribbon)
    .attr("fill", d => color(d.target.index, numCategories))
    .attr("opacity", 0.5)
    .attr("stroke", d => d3.rgb(color(d.target.index, numCategories)).darker())
    .append("title")
    .text(d => `${label(d.source.index)} ${arrow} ${label(d.target.index)}`);
```

The complete code for this example is [available in Github](https://github.com/jarrettmeyer/jarrettmeyer.github.io/blob/master/assets/js/d3-chord-chart.js).

<script src="https://unpkg.com/d3@5.9.2/dist/d3.min.js"></script>
<script src="/assets/js/d3-chord-chart.js"></script>