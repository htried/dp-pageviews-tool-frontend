<template>
  <div id="app">
    <PageSearch @graphData="onGraphSelect" />
    <LineGraph 
      :project="project"
      :country="country"
      :pageID="pageID"
      :startDate="startDate"
      :endDate="endDate"
    />
  </div>
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
