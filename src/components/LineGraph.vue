<template>
    <input type="checkbox" v-model="limitToTop10" @change="renderLineGraph">
    <label for="limitToTop10">Limit to top (up to) 10 cumulative values</label>
    <input type="checkbox" v-model="limitToSelected" @change="renderLineGraph">
    <label for="limitToSelected">Limit to selected country</label>
    <div id="lineGraph"></div>
</template>
  
<script>
import * as d3 from 'd3';
import apr_15_data from './data/2024-04-15.json'
import apr_16_data from './data/2024-04-16.json'
import apr_17_data from './data/2024-04-17.json'
import apr_18_data from './data/2024-04-18.json'
import apr_19_data from './data/2024-04-19.json'
import apr_20_data from './data/2024-04-20.json'
import apr_21_data from './data/2024-04-21.json'
import apr_22_data from './data/2024-04-22.json'
import apr_23_data from './data/2024-04-23.json'
import apr_24_data from './data/2024-04-24.json'

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
      data: [apr_15_data, apr_16_data, apr_17_data, apr_18_data, apr_19_data, apr_20_data, apr_21_data, apr_22_data, apr_23_data, apr_24_data].flat(),
      limitToTop10: false,
      limitToSelected: false
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
    },
    country: {
      hander: 'renderLineGraph',
      immediate: true
    },
    project: {
      hander: 'renderLineGraph',
      immediate: true
    },
    limitToTop10: 'renderLineGraph', // Watch for changes in checkbox state
    limitToSelected: 'renderLineGraph' // Watch for changes in checkbox state
  },
  methods: {
    async fetchData() {
      try {
        // Calculate the number of days between start and end dates
        // const start = new Date(this.startDate);
        // const end = new Date(this.endDate);
        const start = new Date("2024-04-15");
        const end = new Date("2024-04-24");
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
      let filteredData = this.data;
      if (this.limitToSelected) {
        filteredData = filteredData.filter(d => this.country.includes(d.country_code));
      }
      console.log(this.country)

      const totalPageviewsByCountry = new Map();
      filteredData.forEach(d => {
        const country = d.country;
        const pageviews = d.noisy_views;
        totalPageviewsByCountry.set(country, (totalPageviewsByCountry.get(country) || 0) + pageviews);
      });

      // Sort countries by total pageviews in descending order
      const sortedCountries = Array.from(totalPageviewsByCountry.keys()).sort((a, b) => {
        return totalPageviewsByCountry.get(b) - totalPageviewsByCountry.get(a);
      });

      // Create legend items sorted by total pageviews
      var legendItems = sortedCountries.map(country => {
        return { country, totalPageviews: totalPageviewsByCountry.get(country) };
      });
      
      if (this.limitToTop10) {
        let n = 10;
        if (totalPageviewsByCountry.size < 10) {
          n = totalPageviewsByCountry.size;
        }

        // Select top 10 countries
        const topNCountries = sortedCountries.slice(0, n);
        // Filter data for top 10 countries
        filteredData = this.data.filter(d => topNCountries.includes(d.country));
        legendItems = legendItems.slice(0, n)

        if (n == 10) {
          const otherData = this.data.filter(d => !topNCountries.includes(d.country));
          const dailySumByDate = new Map();
          otherData.forEach(d => {
            const date = d.date;
            const pageviews = d.noisy_views;
            dailySumByDate.set(date, (dailySumByDate.get(date) || 0) + pageviews);
          });
          const otherDataAggregated = Array.from(dailySumByDate).map(([date, pageviews]) => ({ date, noisy_views: pageviews, country: 'All other countries' }));
          filteredData = filteredData.concat(otherDataAggregated);

          legendItems = legendItems.concat([{'country': 'All other countries', 'totalPageviews': otherDataAggregated.reduce((total, d) => total + d.noisy_views, 0)}])
        }
      }

      filteredData = filteredData.sort((a, b) => b.date - a.date);

      const margin = { top: 20, right: 20, bottom: 50, left: 80 };
      const width = 1000 - margin.left - margin.right;
      const height = 600 - margin.top - margin.bottom;

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
      const dataByCountry = d3.group(filteredData, d => d.country);

      // Create scales
      const xScale = d3.scaleTime()
                      .domain(d3.extent(filteredData, d => new Date(d.date)))
                      .range([0, width]);

      const yScale = d3.scaleLinear()
                      .domain([0, d3.max(filteredData, d => d.noisy_views)])
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
      dataByCountry.forEach((countryData, country) => {
        svg.append('path')
          .datum(countryData)
          .attr('class', 'line')
          .attr('d', line)
          .attr('fill', 'none')
          .attr('stroke', colorScale(country)) // Set line color based on country
          .attr('stroke-width', 1.5);
      });

      // Create legend container
      const legendContainer = svg.append('foreignObject')
        .attr('x', width)
        .attr('y', 0)
        .attr('width', 300) // Adjust width of the legend container as needed
        .attr('height', height)
        .append('xhtml:div')
        .style('overflow-y', 'auto')
        .style('max-height', height + 'px');

      // Append legend title
      legendContainer.append('text')
        .attr('x', 0)
        .attr('y', 0)
        .attr('dy', '-0.5em')
        .style('text-anchor', 'end')
        .style('font-size', '14px')
        .style('fill', 'black')
        .text('Legend');

      // Append legend items
      legendItems.forEach(item => {
        const legendGroup = legendContainer.append('div')
          .style('display', 'flex')
          .style('align-items', 'center')
          .style('margin', '2px 0');

        // Append colored circle
        legendGroup.append('svg')
          .attr('width', 6)
          .attr('height', 1)
          .style('margin-right', '-295px')
          .append('circle')
          .attr('cx', 10)
          .attr('cy', 10)
          .attr('r', 5) // Set radius of circle
          .attr('fill', colorScale(item.country));

        // Append legend text
        legendGroup.append('span')
          .style('font-size', '12px')
          .style('fill', 'black') // Set legend text color to black
          .text(`${item.country} (Total pageviews: ${item.totalPageviews.toLocaleString()})`);
      });

      // Append X-axis title
      svg.append('text')
        .attr('transform', `translate(${width / 2}, ${height + margin.bottom - 5})`)
        .style('text-anchor', 'middle')
        .style('font-size', '14px')
        .style('fill', 'black')
        .text('Date');

      // Append Y-axis title
      svg.append('text')
        .attr('transform', `rotate(-90) translate(-${height / 2}, ${-margin.left + 20})`)
        // .attr('x', -height / 2)
        // .attr('y', -margin.left + 15)
        .style('text-anchor', 'middle')
        .style('font-size', '14px')
        .style('fill', 'black')
        .text('Page views');
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
  