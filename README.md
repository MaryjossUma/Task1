# Task1. Mandatory Task

We have to face three challenges here:

- Add updated data.
- Create two buttons to swap from inital data to updated.

# Steps

- We will take as starting example _02-pin-location-scale_, let's copy the content from that folder and execute _npm install_.

```bash
npm install
```

- We need coronavirus data updated. We created a new file _dataCommunities.ts_ where we placed initial data and last data.

- Now instead of importing _stats_ we will import _dataCommunites_.

_./src/index.ts_

```diff
- import { stats } from "./stats";
+ import { stats, lastStats, AffectedEntry } from "./dataCommunities";
```

- Let's create a variable to save the data, and change _stast_ into _data_ in _calculateRadiusBasedOnAffectedCases_

_./src/index.ts_

```diff
+ let data = stats;

const calculateRadiusBasedOnAffectedCases = (comunidad: string) => {
- const entry = stats.find((item) => item.name === comunidad);
+ const entry = data.find((item) => item.name === comunidad);

  return entry ? affectedRadiusScale(entry.value) : 0;
};
```

- We created a method to update the circle with the data(inital or lastest).

_./src/index.ts_

```diff
+ const updateChart = (affectedData: AffectedEntry[]) => {
+  data = affectedData;
+  d3.selectAll("circle")
+    .data(latLongCommunities)
+    .transition()
+    .duration(500)
+    .attr("r", (d) => calculateRadiusBasedOnAffectedCases(d.name));
+ };
```

- In the _index.html_ we are going to add the two buttons to can update the map circles.

_./src/index.html_

```diff
<body>
+    <div>
+      <button id="first">Marzo 2020</button>
+      <button id="last">Abril 2021</button>
+    </div>
    <script src="./index.ts"></script>
  </body>
```

- Let's add events listener to this buttons, to update the chart when we click on it.

_./src/index.ts_

```diff
+ document
+  .getElementById("first")
+  .addEventListener("click", function handlFirstResults() {
+    updateChart(stats);
+  });

+ document
+  .getElementById("last")
+  .addEventListener("click", function handlLastResults() {
+    updateChart(lastStats);
+  });
```

- We have very differents ranges in inital data(0-200) and in last data(48000 - 650000).

- For this reason we changed the scale from linear to Threshold, and added a new domain and range.

_./src/index.ts_

```diff
const affectedRadiusScale = d3
-  .scaleLinear()
-  .domain([0, maxAffected])
-  .range([0, 50]); // 50 pixel max radius, we could calculate it relative to width and height
+  .scaleThreshold<number, number>()
+  .domain([20, 50, 200, 100000, 300000, 500000, 700000])
+  .range([5, 10, 15, 20, 30, 40, 50]);
```

-To see better the number of affected in each community we decide to implement a tooltip.

-First we append a div, wit class 'tooltip' and opacity 0 to the body.

_./src/index.ts_

```diff
+ // Tooltip
+ const div = d3
+  .select("body")
+  .append("div")
+  .attr("class", "tooltip")
+  .style("opacity", 0)
```

-Let's add mouse events to the circles, on mouseover, we find the number of affected in data,
then the coords in the event.
-Add a transition into div.tooltip and change opcity to 0.9, to see the dive
-We create a span, where add the information _name: number of affected_ and the coords.
-On mouseout, to hidden the tooltip we change opacity to 0.

_./src/index.ts_

```diff
  .attr("r", (d) => calculateRadiusBasedOnAffectedCases(d.name))
  .attr("cx", (d) => aProjection([d.long, d.lat])[0])
  .attr("cy", (d) => aProjection([d.long, d.lat])[1]);
  .attr("cy", (d) => aProjection([d.long, d.lat])[1])
+  .on("mouseover", function (e: any, datum: any) {
+    const affected = data.find((item) => item.name === datum.name).value;
+    const coords = { x: e.x, y: e.y };
+    div.transition().duration(200).style("opacity", 0.9);
+    div
+      .html(`<span>${datum.name}: ${affected}</span>`)
+      .style("left", `${coords.x}px`)
+      .style("top", `${coords.y - 28}px`);
+  })
+  .on("mouseout", function (datum) {
+    div.transition().duration(500).style("opacity", 0);
+  });
```

-Add a new file, _styles.css_ where apply styles for the tooltip.

_./src/styles.css_

```diff
+ div.tooltip {
+  position: absolute;
+  text-align: center;
+  width: 80px;
+  height: 30px;
+  padding: 2px;
+  font: 12px sans-serif;
+  background: #f7f2cb;
+  border: 0px;
+  border-radius: 8px;
+  pointer-events: none;
+}
```

-Lastly, add this file into _index.html_'

_./src/index.html_

```diff
  <head>
    <link rel="stylesheet" type="text/css" href="./map.css" />
    <link rel="stylesheet" type="text/css" href="./base.css" />
+    <link rel="stylesheet" type="text/css" href="./styles.css" />
  </head>
```
