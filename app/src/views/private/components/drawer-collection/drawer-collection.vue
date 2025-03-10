<template>
	<component
		:is="layoutWrapper"
		v-slot="{ layoutState }"
		v-model:selection="layoutSelection"
		v-model:layout-options="localOptions"
		v-model:layout-query="localQuery"
		v-model:filters="layoutFilters"
		v-model:search-query="searchQuery"
		:collection="collection"
		select-mode
	>
		<v-drawer v-model="internalActive" :title="t('select_item')" @cancel="cancel">
			<template #subtitle>
				<v-breadcrumb :items="[{ name: collectionInfo.name, disabled: true }]" />
			</template>

			<template #title-outer:prepend>
				<v-button class="header-icon" rounded icon secondary disabled>
					<v-icon :name="collectionInfo.icon" :color="collectionInfo.color" />
				</v-button>
			</template>

			<template #actions:prepend><component :is="`layout-actions-${localLayout}`" v-bind="layoutState" /></template>

			<template #actions>
				<search-input v-model="searchQuery" />

				<v-button v-tooltip.bottom="t('save')" icon rounded @click="save">
					<v-icon name="check" />
				</v-button>
			</template>

			<component :is="`layout-${localLayout}`" class="layout" v-bind="layoutState">
				<template #no-results>
					<v-info :title="t('item_count', 0)" :icon="collectionInfo.icon" center />
				</template>

				<template #no-items>
					<v-info :title="t('item_count', 0)" :icon="collectionInfo.icon" center />
				</template>
			</component>
		</v-drawer>
	</component>
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, PropType, ref, computed, toRefs, watch } from 'vue';
import { Filter } from '@directus/shared/types';
import usePreset from '@/composables/use-preset';
import { useCollection } from '@directus/shared/composables';
import { useLayout } from '@/composables/use-layout';
import SearchInput from '@/views/private/components/search-input';

export default defineComponent({
	components: { SearchInput },
	props: {
		active: {
			type: Boolean,
			default: false,
		},
		selection: {
			type: Array as PropType<(number | string)[]>,
			default: () => [],
		},
		collection: {
			type: String,
			required: true,
		},
		multiple: {
			type: Boolean,
			default: false,
		},
		filters: {
			type: Array as PropType<Filter[]>,
			default: () => [],
		},
	},
	emits: ['update:filters', 'update:active', 'input'],
	setup(props, { emit }) {
		const { t } = useI18n();

		const { save, cancel } = useActions();
		const { internalActive } = useActiveState();
		const { internalSelection, onSelect } = useSelection();

		const { collection } = toRefs(props);

		const { info: collectionInfo } = useCollection(collection);
		const { layout, layoutOptions, layoutQuery, searchQuery } = usePreset(collection, ref(null), true);

		// This is a local copy of the layout. This means that we can sync it the layout without
		// having use-preset auto-save the values
		const localLayout = ref(layout.value || 'tabular');
		const localOptions = ref(layoutOptions.value);
		const localQuery = ref(layoutQuery.value);

		const layoutSelection = computed<any>({
			get() {
				return internalSelection.value;
			},
			set(newFilters) {
				onSelect(newFilters);
			},
		});

		const layoutFilters = computed<Filter[]>({
			get() {
				return props.filters;
			},
			set(newFilters) {
				emit('update:filters', newFilters);
			},
		});

		const { layoutWrapper } = useLayout(layout);

		return {
			t,
			save,
			cancel,
			internalActive,
			layoutWrapper,
			layoutSelection,
			layoutFilters,
			localLayout,
			localOptions,
			localQuery,
			collectionInfo,
			searchQuery,
		};

		function useActiveState() {
			const localActive = ref(false);

			const internalActive = computed({
				get() {
					return props.active === undefined ? localActive.value : props.active;
				},
				set(newActive: boolean) {
					localActive.value = newActive;
					emit('update:active', newActive);
				},
			});

			return { internalActive };
		}

		function useSelection() {
			const localSelection = ref<(string | number)[] | null>(null);

			const internalSelection = computed({
				get() {
					if (localSelection.value === null) {
						return props.selection;
					}

					return localSelection.value;
				},
				set(newSelection: (string | number)[]) {
					localSelection.value = newSelection;
				},
			});

			watch(
				() => props.active,
				() => {
					localSelection.value = null;
				}
			);

			return { internalSelection, onSelect };

			function onSelect(newSelection: (string | number)[]) {
				if (newSelection.length === 0) {
					localSelection.value = [];
					return;
				}

				if (props.multiple === true) {
					localSelection.value = newSelection;
				} else {
					localSelection.value = [newSelection[newSelection.length - 1]];
				}
			}
		}

		function useActions() {
			return { save, cancel };

			function save() {
				emit('input', internalSelection.value);
				internalActive.value = false;
			}

			function cancel() {
				internalActive.value = false;
			}
		}
	},
});
</script>

<style lang="scss" scoped>
.layout {
	--layout-offset-top: 0px;
}
</style>
