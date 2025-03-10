<template>
	<ul class="v-list" :class="{ large }">
		<slot />
	</ul>
</template>

<script lang="ts">
import { defineComponent, PropType, toRefs } from 'vue';
import { useGroupableParent } from '@/composables/groupable';

export default defineComponent({
	props: {
		modelValue: {
			type: Array as PropType<(number | string)[]>,
			default: null,
		},
		large: {
			type: Boolean,
			default: false,
		},
		multiple: {
			type: Boolean,
			default: true,
		},
		mandatory: {
			type: Boolean,
			default: true,
		},
	},
	emits: ['update:modelValue'],
	setup(props, { emit }) {
		const { modelValue, multiple, mandatory } = toRefs(props);
		useGroupableParent(
			{
				selection: modelValue,
				onSelectionChange: (newSelection) => {
					emit('update:modelValue', newSelection);
				},
			},
			{
				mandatory,
				multiple,
			}
		);

		return {};
	},
});
</script>

<style scoped>
:global(body) {
	--v-list-padding: 4px 0;
	--v-list-border-radius: var(--border-radius);
	--v-list-max-height: none;
	--v-list-max-width: none;
	--v-list-min-width: 220px;
	--v-list-min-height: none;
	--v-list-color: var(--foreground-normal-alt);
	--v-list-color-hover: var(--foreground-normal-alt);
	--v-list-color-active: var(--foreground-normal-alt);
	--v-list-background-color-hover: var(--background-normal);
	--v-list-background-color-active: var(--background-normal);
}

.v-list {
	position: static;
	display: block;
	min-width: var(--v-list-min-width);
	max-width: var(--v-list-max-width);
	min-height: var(--v-list-min-height);
	max-height: var(--v-list-max-height);
	padding: var(--v-list-padding);
	overflow: auto;
	color: var(--v-list-color);
	line-height: 22px;
	list-style: none;
	border-radius: var(--v-list-border-radius);
}

.large {
	--v-list-padding: 12px;
}

:slotted(.v-divider) {
	max-width: calc(100% - 16px);
	margin: 8px;
}

/* :slotted(*) {
	pointer-events: all;
} */
</style>
