<template>
  <head>
    <title>DP Pageview Visualizer</title>
    <link rel="stylesheet" href="https://tools-static.wmflabs.org/cdnjs/ajax/libs/bootstrap/5.3.3/css/bootstrap.min.css">
    <link href="https://tools-static.wmflabs.org/fontcdn/css?family=Merriweather:400,400italic,700,700italic&subset=latin" rel="stylesheet" type="text/css">
    <link href='https://tools-static.wmflabs.org/fontcdn/css?family=Lato:400,400italic,700,700italic&subset=latin' rel='stylesheet' type='text/css'>
    <link rel="stylesheet" href="./assets/style.css" type='text/css'>
    <link href="./static/favicon.ico" rel="shortcut icon"/>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width = device-width, initial-scale=1, user-scalable = no" />
  </head>
  <div id="origin_bar">
      <div class="boxwidth--1-1 padded--left padded--right">
          <!-- LOCAL -->
          <a href="https://security.wikimedia.org/" class="origin_title"><img src="./assets/Wikimedia-Foundation-logo.svg" alt="Wikimedia Foundation logo" />Wikimedia Privacy</a>
      </div>
  </div>

  <article id="tool_holder">
    <div id="tool_header--holder" class="boxwidth--1-1 padded--left padded--right">
      <header id="tool_header">
          <!-- Tool name -->
          <h1>Differentially private pageview visualizer</h1>
          <div class="meta">
              <div class="disclaimer note">
                  <p>No guarantees are made that this tool will be maintained.</p>
                  <p>This is an experimental tool hosted on <a href="https://wikitech.wikimedia.org/wiki/Portal:Toolforge" target="_blank">Toolforge</a>. No additional personal data is collected by this tool per the Cloud Services <a href="https://wikitech.wikimedia.org/wiki/Wikitech:Cloud_Services_Terms_of_use" target="_blank" rel="noopener">Terms of Use</a>.</p>
              </div>
              <div class="description padded--right">
                  <p>This tool, built at the 2024 Wikimedia Hackathon by <a href="https://meta.wikimedia.org/wiki/User:HTriedman_(WMF)" target="_blank">Hal Triedman</a> and <a href="https://meta.wikimedia.org/wiki/User:MGerlach_(WMF)" target="_blank">Martin Gerlach</a>, provides an example of how we might visualize pageviews for a page, split up by country and day.</p>
                  <p>The source datasets for this tool has been made safer for readers using <a href="https://en.wikipedia.org/wiki/Differential_privacy" target="_blank">differential privacy</a>, and has previously only been available in static CSV format at the following links: <a href="https://analytics.wikimedia.org/published/datasets/country_project_page_historical/" target="_blank">2017-2023</a>, <a href="https://analytics.wikimedia.org/published/datasets/country_project_page/" target="_blank">2023-present</a>.</p>
                  <p>The source code for this tool is available (<a href="https://github.com/htried/dp-pageviews-tool-frontend" target="_blank">frontend</a>, <a href="https://gitlab.wikimedia.org/htriedman/dp-pageviews-tool-backend"  target="_blank">backend</a>) for use under the MIT License at the Wikimedia Foundation Gitlab instance.</p>
              </div>
          </div>
      </header>
    </div>
    <div class="separator"></div>
    <main id="tool_main">
      <section class="boxwidth--1-1 padded--left padded--right">
        <h2>Data grapher</h2>
        <section class="text">
          <p>To use this tool, do the following:</p>
          <ol>
            <li>Enter a project you want to query into the project bar.</li>
            <li>Enter the page title you want to query into the page title bar.</li>
            <li>Enter a start and end date into the start and end date bars.</li>
            <li>Hit the "Graph Data!" button.</li>
          </ol>.
          <p>If you'd like to filter to just the top 10 most viewed countries, hit the checkbox for that limiting. If you'd like to limit to one selected country, type that country into the country search bar, select it, press "Limit to selected country", and hit "Graph data!".</p>
        </section>
      </section>
      <div id="app" class="boxwidth--1-1 padded--left padded--right">
        <PageSearch @graphData="onGraphSelect" />
        <LineGraph 
          :project="project"
          :country="country"
          :pageID="pageID"
          :startDate="startDate"
          :endDate="endDate"
        />
      </div>
    </main>
  </article>
  <footer id="tool_footer">
		<div id="tool_footer-in" class="boxwidth--1-1 padded--left padded--right">
			<p>Experimental tool developed by <span class="tool-author"><a href="https://haltriedman.com" target="_blank">Hal Triedman</a></span> as part of the <a href="https://security.wikimedia.org/" target="_blank">WMF Privacy Engineering</a> team and <a href="https://meta.wikimedia.org/wiki/User:MGerlach_(WMF)" target="_blank">Martin Gerlach</a> as part of the <a href="https://research.wikimedia.org" target="_blank">Research team</a>.</p>
			<ul>
				<li><a href="https://github.com/htried/dp-pageviews-tool-frontend/" target="_blank">View frontend source</a><br /><a href="https://gitlab.wikimedia.org/htriedman/dp-pageviews-tool-backend" target="_blank">View backend source</a></li>
				<li><a href="https://github.com/htried/dp-pageviews-tool-frontend/blob/main/LICENSE" target="_blank">Frontend license</a><br /><a href="https://gitlab.wikimedia.org/htriedman/dp-pageviews-tool-backend/-/blob/main/LICENSE" target="_blank">Backend license</a></li>
			</ul>
		</div>
	</footer>

</template>

<script>
import PageSearch from './components/PageSearch.vue'
import LineGraph from './components/LineGraph.vue'
import '@wikimedia/codex/dist/codex.style.css'
import { ref } from 'vue'

export default {
  name: 'App',
  components: {
    PageSearch,
    LineGraph
  },
  setup() {
    const country = ref('')
    const project = ref('')
    const pageID = ref(0)
    const startDate = ref('')
    const endDate = ref('')

    function onGraphSelect( event ) {
      project.value = event.project;
      country.value = event.country;
      pageID.value = event.pageID;
      startDate.value = event.startDate;
      endDate.value = event.endDate;
    }

    return {
      country,
      project,
      pageID,
      startDate,
      endDate,
      onGraphSelect
    }
  }
}
</script>