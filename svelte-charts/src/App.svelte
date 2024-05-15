<script>
	import * as d3 from 'd3';
 
	let data = [];
	let width = 800;
	let height = 400;
	let margin = { top: 40, right: 20, bottom: 60, left: 50 };
	let color = d3.scaleOrdinal(d3.schemeTableau10);
	let svg;
	let tooltipText = '';
 
	// Load data from CSV
	d3.csv('/data.csv').then((rawData) => {
	   // Count customers per country
	   let countPerCountry = d3.rollup(
		  rawData,
		  v => v.length,
		  d => d.CustomerCountry
	   );
 
	   // Convert map to array and sort
	   data = Array.from(countPerCountry, ([country, count]) => ({ country, count }));
	   data.sort((a, b) => d3.descending(a.count, b.count));
	   
	   // Create the chart
	   createChart(data);
	});
 
	function createChart(data) {
	   svg = d3.select('#chart')
		  .append('svg')
		  .attr('width', width)
		  .attr('height', height);
 
	   // Create scales
	   const x = d3.scaleBand()
		  .domain(data.map(d => d.country))
		  .range([margin.left, width - margin.right])
		  .padding(0.1);
 
	   const y = d3.scaleLinear()
		  .domain([0, d3.max(data, d => d.count)]).nice()
		  .range([height - margin.bottom, margin.top]);
 
	   // Create axes
	   const xAxis = g => g
		  .attr('transform', `translate(0,${height - margin.bottom})`)
		  .call(d3.axisBottom(x).tickSizeOuter(0))
		  .selectAll("text")
			 .attr("transform", "rotate(-45)")
			 .style("text-anchor", "end");
 
	   const yAxis = g => g
		  .attr('transform', `translate(${margin.left},0)`)
		  .call(d3.axisLeft(y).ticks(10, "d"));
 
	   // Create bars
	   svg.append('g')
		  .selectAll('rect')
		  .data(data)
		  .join('rect')
		  .attr('x', d => x(d.country))
		  .attr('y', d => y(d.count))
		  .attr('height', d => y(0) - y(d.count))
		  .attr('width', x.bandwidth())
		  .attr('fill', d => color(d.country))
		  .on('click', (event, d) => {
			 tooltipText = `Country: ${d.country}, Customers: ${d.count}`;
		  });
 
	   svg.append('g').call(xAxis);
	   svg.append('g').call(yAxis);
 
	   // Add title
	   svg.append('text')
		  .attr('x', width / 2)
		  .attr('y', margin.top / 2)
		  .attr('text-anchor', 'middle')
		  .style('font-size', '16px')
		  .text('Number of Customers by Country');
	}
 </script>
 
 <div id="chart"></div>
 {#if tooltipText}
	<p>{tooltipText}</p>
 {/if}
 