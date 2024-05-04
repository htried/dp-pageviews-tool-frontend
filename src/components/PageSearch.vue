<template>
  <div>
    <ProjectSelect @project-selected="onProjectSelected" />
    <CountrySelect @country-selected="updateCountry" />
    <StartDate @start-date="updateStartDate" />
    <EndDate @end-date="updateEndDate" />
    <p>Page ID: {{ selection }}</p>
    <cdx-lookup
      v-model:selected="selection"
      :menu-items="menuItems"
      :menu-config="menuConfig"
      aria-label="Page lookup with fetched results"
      @input="onInput"
    >
      <template #no-results> No results found. </template>
    </cdx-lookup>
  </div>
  <cdx-button @click="graphData">Graph data!</cdx-button>
</template>

<script>
import { defineComponent, ref } from 'vue'
import { CdxLookup, CdxButton } from '@wikimedia/codex'
import ProjectSelect from './ProjectSelect.vue'
import CountrySelect from './CountrySelect.vue'
import StartDate from './StartDate.vue'
import EndDate from './EndDate.vue'

export default defineComponent({
  name: 'PageSearch',
  components: { 
    CdxLookup,
    CdxButton,
    ProjectSelect,
    CountrySelect,
    StartDate,
    EndDate
  },
  emits: [
    'graphData'
  ],
  setup(props, {emit}) {
    const selection = ref(null)
    const menuItems = ref([])
    const currentSearchTerm = ref('')
    const selectedCountry = ref('')
    const selectedProject = ref('')
    const startDate = ref('')
    const endDate = ref('')

    function onProjectSelected( project ) {
      selectedProject.value = project;
    }

    function updateCountry( country ) {
      selectedCountry.value = country;
    }

    function updateStartDate( newStartDate ) {
      startDate.value = newStartDate;
    }

    function updateEndDate( newEndDate ) {
      endDate.value = newEndDate;
    }

    /**
     * Get search results.
     *
     * @param {string} searchTerm
     *
     * @return {Promise}
     */
    function fetchResults(searchTerm) {
      const params = new URLSearchParams({
        q: searchTerm,
        limit: '10'
      })
      // TODO make this selectable
      return fetch(`https://${selectedProject.value}/w/rest.php/v1/search/title?${params.toString()}`).then(
        (response) => response.json()
      )
    }

    /**
     * Handle lookup input.
     *
     * @param {string} value
     */
    function onInput(value) {
      // Internally track the current search term.
      currentSearchTerm.value = value

      // Do nothing if we have no input.
      if (!value) {
        menuItems.value = []
        return
      }

      fetchResults(value)
        .then((data) => {
          // Make sure this data is still relevant first.
          if (currentSearchTerm.value !== value) {
            return
          }

          // Reset the menu items if there are no results.
          if (!data.pages || data.pages.length === 0) {
            menuItems.value = []
            return
          }

          // Build an array of menu items.
          const results = data.pages.map((result) => {
            return {
              label: result.title,
              value: result.id,
              description: result.description
            }
          })

          // Update menuItems.
          menuItems.value = results
        })
        .catch(() => {
          // On error, set results to empty.
          menuItems.value = []
        })
    }

    function graphData() {
      emit('graphData', {
        'country': selectedCountry.value,
        'project': selectedProject.value,
        'pageID': selection.value,
        'startDate': startDate.value,
        'endDate': endDate.value
      })
    }

    const menuConfig = {
      visibleItemLimit: 6
    }

    return {
      selection,
      onProjectSelected,
      updateCountry,
      updateStartDate,
      updateEndDate,
      menuItems,
      menuConfig,
      onInput,
      graphData
    }
  }
})
</script>
