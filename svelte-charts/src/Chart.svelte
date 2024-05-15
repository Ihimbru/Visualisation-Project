<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let data = [];
    let countByCountry = [];
  
    onMount(async () => {
      const csvData = await d3.csv('data.csv');
      const countryMap = new Map();
  
      // Count the number of customers per country
      csvData.forEach(({ CustomerCountry }) => {
        if (countryMap.has(CustomerCountry)) {
          countryMap.set(CustomerCountry, countryMap.get(CustomerCountry) + 1);
        } else {
          countryMap.set(CustomerCountry, 1);
        }
      });
  
      countByCountry = Array.from(countryMap, ([Country, Count]) => ({ Country, Count }));
    });
  
    // Helper function to generate color dynamically
    const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
  </script>
  
  <!-- Bar Chart -->
  <svg width="800" height="500">
    {#each countByCountry as { Country, Count }, i}
      <rect x={i * 30} y={500 - Count * 10} width="20" height={Count * 10} fill={colorScale(i)} />
      <text x={i * 30 + 10} y={490} fill="black" font-size="12px" text-anchor="middle">{Country}</text>
    {/each}
  </svg>
  
  <!-- Pie Chart -->
  <svg width="800" height="800">
    <g transform="translate(400, 400)">
      {#each d3.pie().value(d => d.Count)(countByCountry) as {data, startAngle, endAngle, index}}
        <path d={d3.arc().innerRadius(0).outerRadius(200).startAngle(startAngle).endAngle(endAngle)()} fill={colorScale(index)} />
        <text transform={`translate(${d3.arc().innerRadius(0).outerRadius(100).centroid({startAngle, endAngle})})`} text-anchor="middle" fill="white">
          {data.Country}
        </text>
      {/each}
    </g>
  </svg>
  