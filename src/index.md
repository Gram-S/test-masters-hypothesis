---
toc: false
---
<!-- TO DO:
Make the graph directed
Impliment some form of time axis
People on the nodes

-->

# Phylogeny of Mathematics Doctoral Advisors

<img src="Meme.png" width=700 />
<figcaption>"The Simpsons" Season 9 Episode 6, "Bart Star". Photos from wikipedia.</figcaption>



When we study mathematics history, we often notice that while studying an individual, their Doctoral advisor is familiar. Perhaps we've studied or heard of them before? That is to imply, someone is more likely to be notable (documented) in history if their doctorial advisor is notable. Here we represent the history of math and computer science (fairly interconnected disciplines) as a network, with people as nodes and edges representing the advisor-advisee relationship (binary choice, either they have it or they don't). Which leads us to the following hypothesis. 

## Hypothesis

*A random sampling of the historical network of mathematicians and computer scientists from 1800-2000 will contain many connected subgraphs of about 3-5 components, rather than being completely sparse.*

## Data
<!-- Original import-->
```js
import {database} from "./components/variables.js";
display(Inputs.table(database, {select: false}));
```

Data is made from webscraping wikipedia using the wikimedia foundation's API. Random sampling is done mainly due concern that any source which attempts to provide a valid timeline will be subject to significant bias in support of the hypothesis. This is due to our original hypothesis assuming that advisor-advisee relationships make people more likely to be documented, and thus more likely to appear in the same source. Some names are referenced from wikipedia's various computer science history timelines.  

<!-- Network graph -->
```js
// One edge per (advisor -> student); advisors are ";"-separated in the CSV
const links = database.flatMap((d) =>
  (d.doctoral_advisor ?? "")
    .split(";")
    .map((a) => a.trim())
    .filter(Boolean)
    .map((a) => ({source: a, target: d.wikipedia_title || d.query_name}))
);

// Every person gets a node, with or without an advisor
const people = database.map((d) => d.wikipedia_title || d.query_name);
const nodes = Array.from(new Set([...people, ...links.flatMap((l) => [l.source, l.target])]), (id) => ({id}));

// Run the force layout to completion up front (no animation)
d3.forceSimulation(nodes)
  .force("link", d3.forceLink(links).id((d) => d.id).distance(40))
  .force("charge", d3.forceManyBody().strength(-80))
  .force("x", d3.forceX().strength(0.05))
  .force("y", d3.forceY().strength(0.05))
  .stop()
  .tick(300);

display(
  Plot.plot({
    width,
    aspectRatio: 1,
    inset: 20,
    axis: null,
    marks: [
      Plot.link(links, {
        x1: (d) => d.source.x,
        y1: (d) => d.source.y,
        x2: (d) => d.target.x,
        y2: (d) => d.target.y,
        strokeOpacity: 0.4
      }),
      Plot.dot(nodes, {x: "x", y: "y", r: 3, title: "id"}),
      Plot.text(nodes, {x: "x", y: "y", text: "id", dy: -8, fontSize: 9})
    ]
  })
);
```
