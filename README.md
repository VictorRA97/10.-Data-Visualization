# Task-1

In this task we are going to represent the number of cases of coronavirus by autonomous community in Spain. The size of the circle increases as there are more cases of coronavirus.


# Initial Covid Representation
![image](https://user-images.githubusercontent.com/94138220/180602552-da62b62d-5db4-474a-abd2-b52e12fe4d44.png)

# Last Covid Representation
![image](https://user-images.githubusercontent.com/94138220/180602577-836433f4-fc65-4ed2-bc66-6df22c01d35b.png)

# Steps

```bash
npm install
```

- We are going to install the necessary packages to work with TopoJSON:

```bash
npm install topojson-client --save
```

```bash
npm install @types/topojson-client --save-dev
```

```bash
npm install topojson --save
```

```bash
npm install @types/topojson --save-dev
```

```bash
npm install d3-composite-projections --save
```

- We import all the data with which we are going to work: TopoJSON, the file of Spain, the file of the autonomous communities, the projections of the Canary Islands and the covid data with which we will work.

- In: _./src/index.ts_

```typescript
import * as d3 from "d3";
import * as topojson from "topojson-client";
const spainjson = require("./spain.json");
const d3Composite = require("d3-composite-projections");
import { latLongCommunities } from "./communities";
import { statsIni,statsLast,ResultEntry } from "./covid";
```
- The following function gives us the radius of the circumference that represents the cases of covid in the autonomous community:

```typescript
const calculateRadiusBasedOnAffectedCases = (comunidad: string,data: ResultEntry[]) => {
  const entry = data.find(item => item.name === comunidad);
  const affectedRadiusScale = d3
  .scaleLinear()
  .domain([0, 10000])
  .range([5, 40]); 
  return entry ? affectedRadiusScale(entry.value) : 0;
};
```

-We continue with the settings and configurations for the representation of the map of Spain and its Covid cases:

```typescript
const svg = d3
  .select("body")
  .append("svg")
  .attr("width", 1024)
  .attr("height", 800)
  .attr("style", "background-color: #FBFAF0");

const aProjection = d3Composite
  .geoConicConformalSpain()
  // Let's make the map bigger to fit in our resolution
  .scale(3300)
  // Let's center the map
  .translate([500, 400]);

const geoPath = d3.geoPath().projection(aProjection);
const geojson = topojson.feature(spainjson, spainjson.objects.ESP_adm1);

svg
  .selectAll("path")
  .data(geojson["features"])
  .enter()
  .append("path")
  .attr("class", "country")
  // data loaded from json file
  .attr("d", geoPath as any);

svg
  .selectAll("circle")
  .data(latLongCommunities)
  .enter()
  .append("circle")
  .attr("class", "affected-marker")
  .attr("r", d => calculateRadiusBasedOnAffectedCases(d.name,statsIni))
  .attr("cx", d => aProjection([d.long, d.lat])[0])
  .attr("cy", d => aProjection([d.long, d.lat])[1]);
  ```
