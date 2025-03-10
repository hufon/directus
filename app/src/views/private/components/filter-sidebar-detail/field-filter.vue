<template>
	<div class="field-filter">
		<div class="header">
			<div v-tooltip="filter.field.split('.').join(' → ')" class="name">
				<span v-if="filter.field.includes('.')" class="relational-indicator">•</span>
				{{ name }}
			</div>
			<v-menu show-arrow :disabled="disabled">
				<template #activator="{ toggle }">
					<div v-tooltip.top="t('change_advanced_filter_operator')" class="operator" @click="toggle">
						<span>{{ t(`operators.${activeOperator}`) }}</span>
						<v-icon name="expand_more" />
					</div>
				</template>

				<v-list>
					<v-list-item
						v-for="operator in filterOperators"
						:key="operator"
						:active="operator === activeOperator"
						clickable
						@click="activeOperator = operator"
					>
						<v-list-item-content>{{ t(`operators.${operator}`) }}</v-list-item-content>
					</v-list-item>
				</v-list>
			</v-menu>
			<div class="spacer" />
			<v-icon
				v-tooltip.left="t('delete_advanced_filter')"
				class="remove"
				name="close"
				clickable
				@click="$emit('remove')"
			/>
		</div>
		<div class="field">
			<filter-input v-model="value" :field="field" :type="field.type" :operator="activeOperator" :disabled="disabled" />
		</div>
	</div>
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, PropType, computed } from 'vue';
import { Filter } from '@directus/shared/types';
import { useFieldsStore } from '@/stores';
import { getFilterOperatorsForType } from '@directus/shared/utils';
import FilterInput from './filter-input.vue';

export default defineComponent({
	components: { FilterInput },
	props: {
		filter: {
			type: Object as PropType<Filter>,
			required: true,
		},
		collection: {
			type: String,
			required: true,
		},
		disabled: {
			type: Boolean,
			default: false,
		},
	},
	emits: ['remove', 'update'],
	setup(props, { emit }) {
		const { t } = useI18n();

		const fieldsStore = useFieldsStore();

		const activeOperator = computed({
			get() {
				return props.filter.operator;
			},
			set(newOperator) {
				emit('update', { operator: newOperator });
			},
		});

		const value = computed({
			get() {
				return props.filter.value;
			},
			set(newValue) {
				emit('update', { value: newValue });
			},
		});

		const name = computed<string>(() => {
			return getNameForFieldKey(props.filter.field);
		});

		const field = computed(() => getFieldForKey(props.filter.field));

		const filterOperators = computed(() => {
			return getFilterOperatorsForType(field.value.type);
		});

		return { t, activeOperator, value, name, field, filterOperators };

		function getFieldForKey(fieldKey: string) {
			return fieldsStore.getField(props.collection, fieldKey);
		}

		function getNameForFieldKey(fieldKey: string) {
			return getFieldForKey(fieldKey)?.name;
		}
	},
});
</script>

<style lang="scss" scoped>
.field-filter {
	.header {
		display: flex;
		align-items: center;
		margin-bottom: 4px;
	}

	.name,
	.operator {
		overflow: hidden;
		white-space: nowrap;
	}

	.name {
		position: relative;
		margin-right: 8px;
		overflow: visible;
		text-overflow: ellipsis;

		.relational-indicator {
			position: absolute;
			top: -1px;
			left: -10px;
			color: var(--foreground-subdued);
		}
	}

	.operator {
		display: flex;
		align-items: center;
		color: var(--primary);
		cursor: pointer;

		span {
			flex-grow: 1;
			overflow: hidden;
			white-space: nowrap;
			text-overflow: ellipsis;
		}
	}

	.spacer {
		flex-grow: 1;
	}

	.remove {
		--v-icon-color: var(--foreground-subdued);

		flex-grow: 0;
		flex-shrink: 0;
		opacity: 0;

		&:hover {
			--v-icon-color: var(--danger);
		}
	}

	&:hover {
		.header .remove {
			opacity: 1;
		}
	}
}
</style>
