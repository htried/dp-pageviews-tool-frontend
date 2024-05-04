<template>
  <div>
    <p class="cdx-docs-demo-text">Selected country value: {{ selection }}</p>

    <cdx-lookup
      v-model:selected="selection"
      :menu-items="menuItems"
      :menu-config="menuConfig"
      aria-label="Lookup countries"
      @input="onInput"
      @update:selected="$emit( 'country-selected', $event )"
    />
  </div>
</template>

<script>
import { defineComponent, ref } from 'vue'
import { CdxLookup } from '@wikimedia/codex'
import countries from './countries.json'

export default defineComponent({
  name: 'CountryLookup',
  components: { CdxLookup },
  emits: [
    'country-selected'
  ],
  setup() {
    const selection = ref(null)
    const menuItems = ref([])

    const menuConfig = {
      boldLabel: true
    }

    /**
     * Filter items on input.
     *
     * @param {string} value
     */
    function onInput(value) {
      if (value) {
        menuItems.value = countries.filter(
          (item) => item.label.toLowerCase().includes(value.toLowerCase()) || item.description.toLowerCase().includes(value.toLowerCase())
        )
        
        if (menuItems.value.length > 10) {
          menuItems.value = menuItems.value.slice(0, 10)
        }
      }
    }

    return {
      selection,
      menuItems,
      menuConfig,
      onInput
    }
  }
})
</script>
