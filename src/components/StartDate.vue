<template>
	<div>
		<p>Start date: {{ inputValue }}</p>

		<cdx-text-input
			v-model="inputValue"
			aria-label="start date input"
			input-type="date"
			@update:modelValue="$emit( 'start-date', $event )"
		/>
	</div>
</template>

<script>
import { defineComponent, ref } from 'vue';
import { CdxTextInput } from '@wikimedia/codex';

export default defineComponent( {
	name: 'StartDate',
	components: { CdxTextInput },
	emits: [
		'start-date'
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