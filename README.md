# Live Coding Pollens Svelte D3 - Cheatsheet

## Start the project 
### Prerequis
node 10+ / npm 6

Mode présentation IntelliJ : ctrl + ` (police 24)

### Installation 
```
npx degit sveltejs/template pollens-demo
```
Changer les versions dans `package.json`
```
   "rollup": "^1.2.2",
   "rollup-plugin-commonjs": "^9.2.0",
   "rollup-plugin-json": "^3.1.0",
   "rollup-plugin-node-resolve": "^4.0.1",
   "rollup-plugin-svelte": "^5.0.3",
   "rollup-plugin-terser": "^4.0.4",
   "sirv-cli": "^0.2.3",
   "svelte": "^3.0.0-beta.7"
```

```
npm install
npm run dev
```

### `rollup.config.js`
* supprimer de la conf svelte dans `rollup.config.js` :
skipIntroByDefault et nestedTransitions
* Ajouter 
```
import json from 'rollup-plugin-json';

json(),
```

les composants sotn en .svelte et plus .html

## Initialisation 
Hello World

* ajout des données Json de Pollens.fr

## Découverte de Svelte
* Modification de titre H1
* Import de json avec rollup-json
```
import pollens from '../assets/pollens.json';
```
* Déclaration de la propriété datasets, initialiser avec []

* Affichage avec le template each et un <ul>
```
{#each datasets as dataset} 
{:else}
{/each}
```
* passage en props au composant App avec export

* Transformation en Select avec le destructuring et l'index

* Composant Svelte viz/Viz.svelte
```
import Viz from './viz/Viz.svelte';
```

* Attribut et squelette :
```
<h2> avec le titre et les data
<svg width height>
```


* Récupération de la sélection, ajout d'une directive
```
let index = 0;
let currentData = {};
const selectData = () => currentData = datasets[index];
selectData();

bind:value={index} on:change={selectData}
```
* Passage des arguments au composant


## D3 - Part 1 : LineChart
* Ajout des modules D3 
```
npm install d3-axis d3-scale d3-time-format d3-shape d3-selection
```

* Normes D3 
```
import {scaleTime, scaleLinear} from 'd3-scale';
import {axisBottom, axisLeft} from 'd3-axis';
import {timeFormat, timeParse} from 'd3-time-format';
import {select} from 'd3-selection';
import {line} from 'd3-shape';
import {afterUpdate} from 'svelte'; 

const margin = {top: 20, right: 25, bottom: 20, left: 35};
const red = '#b21e3e';

const xScale = scaleTime()
                .domain([new Date(2000, 0, 1), new Date(2000, 11, 31)])
                .range([margin.left, width - margin.right]);

const yScale = scaleLinear()
                .domain([0, 5])
                .range([height - margin.bottom, margin.top]);

const xAxis = axisBottom().scale(xScale)
    .tickFormat(timeFormat('%b'));
const yAxis = axisLeft().scale(yScale);
const parse = timeParse("%Q");
```

* Ajout des Axes au SVG et translation
```
afterUpdate(() => {
    select('g[ref="xAxis"]').call(xAxis);
    select('g[ref="yAxis"]').call(yAxis);
});

<g>
  <g ref='xAxis' transform={`translate(0, ${height - margin.bottom})`} />
  <g ref='yAxis' transform={`translate(${margin.left}, 0)`} />
</g>
```

* Affichage des données
```
const getDay = timestamp => {
    const day = parse(timestamp);
    day.setYear(2000);
    return day;
};

const path = line().x(d => xScale(getDay(d.date)))
                   .y(d => yScale(d.level));

afterUpdate(() => {
    cleanedData = data.hasOwnProperty('data')
                      ? data.data
                      : [];
});

<path d='{path(cleanedData)}' fill='none' stroke={red} strokeWidth='2' />

```

* Tri des données
```
cleanedData = data.hasOwnProperty('data')
                  ? data.data.sort((a,b) => a.date - b.date)
                  : [];
```


* Ajout d'une légende , mettre les styles dans le composant !
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
	width: min;
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


## D3 - Part 2 : Radial bar chart
* copier le linechart, nettoyer, supprimer :
```
svg > path + g

Axis, Path, scales, getDay
```

* Définir l'angle correspondant à un jour et l'échelle des rayons
```
const dayAngle = 2 * Math.PI / 365;

const radiusScale = scaleLinear()
                        .domain([0,5])
                        .range([0, width/2 - margin.left]);
```

* Transformer les données en arc et afficher dans le svg 
```
slices = cleanedData.map((d,i) => {
        const shape =  arc().startAngle(i * dayAngle)
                            .endAngle((i+1) * dayAngle)
                            .innerRadius(radiusScale(0))
                            .outerRadius(radiusScale(d.level))();

        return {shape, fill:red};
    })
    
<g class="data">
 {#each slices as {shape, fill}}
      <path d="{shape}" fill="{fill}"></path>
 {/each}
</g>   
```

* Décaler le contenu et supprimer le paramètre de hauteur
```
<g transform="translate({width/2},{width/2})">
```

* Ajouter les mois (tranches + noms)
```
const months = ['jan', 'fev', 'mars', "avr", 'mai', 'juin', 'juil', 'aout', 'sep', 'oct', 'nov', 'dec'];
const monthAngle = 2 * Math.PI / 12;
const monthSlices = months.map((d,i) => {
       const shape =  arc().startAngle(i * monthAngle)
                           .endAngle((i+1) * monthAngle)
                           .innerRadius(radiusScale(0))
                           .outerRadius(radiusScale(5))();

       return {shape, name: d};
});

<g class="months">
    {#each monthSlices as {shape, name}, i}
        <path id="month{i}" d="{shape}"></path>
        <text>
            <textPath href="#month{i}" startOffset="7%">{name}</textPath>
        </text>
    {/each}
</g>

.months path {
    fill:none;
    stroke: lightgray;
    stroke-width: 1px;
    stroke-dasharray: 5 2;
}
```

* Ajout de l'échelle des pollens
```
const levels = [1, 2, 3, 4, 5];

{#each levels as level }
    <circle cx="0" cy="0" r="{radiusScale(level)}"/>
    <text x="0" y={-radiusScale(level)}>{level}</text>
{/each}

.levels circle {
    fill: none;
    stroke: lightblue;
    stroke-width: 1px;
    stroke-dasharray: 5;
}
```

* Touche finale : la couleur avec Chroma.js
```
npm install chroma-js

import chroma from 'chroma-js';
const colors = chroma.scale(['green', 'orange', 'red']);
const colorScale = scaleLinear()
                .domain([0, 5]);
                
fill: colors(colorScale(d.level))
              
```

* Montrer les 2 viz en même temps
```
.group-viz {
    display: flex;
    flex-wrap: wrap;
}
```

## D3 - Bundle !
* Modifier le nom du bundle js et css dans `rollup.config.js`.
* Créer un conteneur et l'indiquer en target de l'App :
```
document.querySelector('div#viz')
```

* Ajouter une section et la même div dans la presentation Reveal

* Custom.css de REveal doit avoir :
```
.reveal #viz {
    font-family: 'Roboto', sans-serif;
    font-size: 0.8rem;
    color: black;
    background: whitesmoke;
}
.reveal #viz h2, .reveal #viz  h1 {
    color: black;
}
.group-viz {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-evenly;
}
```
