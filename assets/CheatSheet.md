## Cheatsheet copier-coller

* Install D3 et chroma
```
npm install d3-axis d3-scale d3-time-format d3-shape d3-selection chroma-js
```

* Normes D3
```js
import {scaleTime, scaleLinear} from 'd3-scale';
import {axisBottom, axisLeft} from 'd3-axis';
import {timeFormat, timeParse} from 'd3-time-format';
import {select} from 'd3-selection';
import {line} from 'd3-shape';
import {afterUpdate} from 'svelte';

// Marges
const margin = {top: 20, right: 20, bottom: 20, left: 25};
const red = '#b21e3e';

// Echelles et Axes
const xScale = scaleTime()
                .domain([new Date(2000, 0, 1),new Date(2000, 11, 31) ])
                .range([margin.left, width - margin.right]);

const yScale = scaleLinear()
                .domain([0, 5])
                .range([height - margin.bottom, margin.top]);

// Abscisse avec juste le mois
const xAxis = axisBottom().scale(xScale)
                .tickFormat(timeFormat('%b'));

const yAxis = axisLeft().scale(yScale);

// Parsing des timestamps
const parse = timeParse('%Q');



afterUpdate(() => {
   // Génération des axes 
   select('g[ref="xAxis"]').call(xAxis);
   select('g[ref="yAxis"]').call(yAxis);
});


<g>
  <g ref="xAxis" transform={`translate(0, ${height - margin.bottom})`}></g>
  <g ref="yAxis" transform={`translate(${margin.left}, 0)`}></g>
</g>
```

* Légende
```
<div class="legend">
     <span>Niveau de pollen :</span>
    <span>0: 😃</span>
    <span>1: 😐</span>
    <span>2: 🤧</span>
    <span>3: 😰</span>
    <span>4: 😱</span>
    <span>5: 💀</span>
</div>

.container {
	width: min-content;
	width: -moz-min-content;
	width: -webkit-min-content;
}
.container .legend {
	width: 100%;
	display: flex;
	justify-content: space-evenly;
	margin: 1rem 0;
}
```

* Mois 
```
const levels =[1,2,3,4,5];
const months = ['jan', 'fev', 'mars', 'avril', 'mai', 'juin',
'juil', 'août', 'sep', 'oct', 'nov', 'dec'];


.months path {
    fill: none;
    stroke: lightgray;
    stroke-width: 1px;
    stroke-dasharray: 5 2;
}
.levels circle {
    fill: none;
    stroke: lightblue;
    stroke-width: 1px;
    stroke-dasharray: 5;
}
```

* Chroma
```
import chroma from 'chroma-js';

const colors = chroma.scale(['green', 'orange', 'red']);
const colorScale = scaleLinear().domain([0,5]);
colors(colorScale(d.level)
```