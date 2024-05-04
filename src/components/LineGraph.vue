<template>
    <div id="lineGraph"></div>
</template>
  
  <script>
  import * as d3 from 'd3';
  import apr_15_data from './data/2024-04-15.json'
  import apr_16_data from './data/2024-04-16.json'
  import apr_17_data from './data/2024-04-17.json'
  
  export default {
    name: 'LineGraph',
    props: {
      country: String,
      project: String,
      pageID: Number,
      startDate: String,
      endDate: String
    },
    data() {
      return {
        // data: []
        data: [apr_15_data, apr_16_data, apr_17_data].flat()
      };
    },
    mounted() {
      this.fetchData();
    },
    watch: {
      // Watch for changes in props
      startDate: {
        handler: 'fetchData', // Call fetchData method when startDate prop changes
      },
      endDate: {
        handler: 'fetchData', // Call fetchData method when endDate prop changes
        immediate: true // Call handler immediately when component is mounted
      }
    },
    methods: {
      async fetchData() {
        try {
          // Calculate the number of days between start and end dates
          const start = new Date(this.startDate);
          const end = new Date(this.endDate);
          const dateRange = [];
          let currentDate = new Date(start);

          while (currentDate <= end) {
            dateRange.push(new Date(currentDate));
            currentDate.setDate(currentDate.getDate() + 1);
          }

          // Make multiple API requests, one for each day
          // const promises = dateRange.map(async date => {
            // remote try
            // const params = new URLSearchParams({
            //   country: this.country,
            //   project: this.project,
            //   pageID: this.pageID,
            //   date: date.toISOString().split('T')[0] // Format date as 'YYYY-MM-DD'
            // })
            // const response = await fetch(`YOUR_API_ENDPOINT/${params.toString()}`);
            // return response.data;

            // local try
            // console.log(`data/${date.toISOString().split('T')[0]}`)
            // const response = await fetch(`/Users/haltriedman/code/dp-pageviews-tool-frontend/src/components/data/${date.toISOString().split('T')[0]}`);
            // const dayResponse = await response.json()
            // console.log(dayResponse)
            // return dayResponse
          // });

          // Wait for all requests to complete
          // const responseData = await Promise.all(promises);

          // Concatenate all data from responses
          // this.data = responseData.reduce((acc, data) => acc.concat(data), []);

          // local try
          // Combine data from all JSON files
          // this.data = responseData.flat(); // Flatten array of arrays

          // Render line graph
          this.renderLineGraph();
        } catch (error) {
          console.error('Error fetching data:', error);
        }
      },
      renderLineGraph() {
        const margin = { top: 20, right: 20, bottom: 50, left: 50 }; // Increased bottom margin to accommodate legend
        const width = 600 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        // Remove any existing SVG elements
        d3.select('#lineGraph svg').remove();

        // Create SVG element
        const svg = d3.select('#lineGraph')
                      .append('svg')
                      .attr('width', width + margin.left + margin.right)
                      .attr('height', height + margin.top + margin.bottom)
                      .append('g')
                      .attr('transform', `translate(${margin.left},${margin.top})`);

        // Group data by country
        const dataByCountry = d3.group(this.data, d => d.country);

        // Create scales
        const xScale = d3.scaleTime()
                        .domain(d3.extent(this.data, d => new Date(d.date)))
                        .range([0, width]);

        const yScale = d3.scaleLinear()
                        .domain([0, d3.max(this.data, d => d.noisy_views)])
                        .nice() // Make y-axis nice
                        .range([height, 0]);

        // Create axes
        const xAxis = d3.axisBottom(xScale);
        const yAxis = d3.axisLeft(yScale);

        // Append x-axis
        svg.append('g')
          .attr('transform', `translate(0,${height})`)
          .call(xAxis)
          .selectAll('text')
          .style('fill', 'black'); // Set x-axis text color to black

        // Append y-axis
        svg.append('g')
          .call(yAxis)
          .selectAll('text')
          .style('fill', 'black'); // Set y-axis text color to black

        // Define line generator
        const line = d3.line()
                      .x(d => xScale(new Date(d.date)))
                      .y(d => yScale(d.noisy_views));

        // Define color scale for lines
        const colorScale = d3.scaleOrdinal()
                            .domain([...dataByCountry.keys()])
                            .range(d3.schemeCategory10); // Generate 10 different colors

        // Render line for each country
        let yOffset = 0;
        dataByCountry.forEach((countryData, country) => {
          svg.append('path')
            .datum(countryData)
            .attr('class', 'line')
            .attr('d', line)
            .attr('fill', 'none')
            .attr('stroke', colorScale(country))
            .attr('stroke-width', 1.5);

          // Append legend
          svg.append('text')
            .attr('x', width + 20) // Adjust x-coordinate for legend text
            .attr('y', yOffset) // Set vertical position based on yOffset
            .attr('dy', '.35em')
            .style('font-size', '12px')
            .style('fill', colorScale(country))
            .text(country);

          // Update yOffset for next legend item
          yOffset += 20; // Increase yOffset for spacing
        });

        // Append legend title
        svg.append('text')
          .attr('x', width)
          .attr('y', 0)
          .attr('dy', '-0.5em')
          .style('text-anchor', 'end')
          .style('font-size', '14px')
          .text('Legend');
      }
    }
  }
  </script>
  
  <style scoped>
  #graphArea {
    width: 100%;
    height: 400px; /* Adjust height as needed */
  }
  </style>
  