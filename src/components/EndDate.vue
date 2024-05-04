<template>
	<div>
		<p>End date: {{ inputValue }}</p>

		<cdx-text-input
			v-model="inputValue"
			aria-label="end date input"
			input-type="date"
            @update:modelValue="$emit( 'end-date', $event )"
		/>
	</div>
</template>

<script>
import { defineComponent, ref } from 'vue';
import { CdxTextInput } from '@wikimedia/codex';

export default defineComponent( {
	name: 'EndDate',
	components: { CdxTextInput },
    emits: [
		'end-date'
	],
	setup() {
		// Set up a reactive reference to track the input value.
		const inputValue = ref( '' );

		return {
			inputValue
		};
	}
} );
</script>