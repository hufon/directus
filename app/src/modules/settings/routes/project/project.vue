<template>
	<private-view :title="t('settings_project')">
		<template #headline>{{ t('settings') }}</template>
		<template #title-outer:prepend>
			<v-button class="header-icon" rounded disabled icon secondary>
				<v-icon name="public" />
			</v-button>
		</template>

		<template #actions>
			<v-button v-tooltip.bottom="t('save')" icon rounded :disabled="noEdits" :loading="saving" @click="save">
				<v-icon name="check" />
			</v-button>
		</template>

		<template #navigation>
			<settings-navigation />
		</template>

		<div class="settings">
			<v-form v-model="edits" :initial-values="initialValues" :fields="fields" :primary-key="1" />
		</div>

		<template #sidebar>
			<project-info-sidebar-detail />
		</template>
	</private-view>
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, ref, computed } from 'vue';
import SettingsNavigation from '../../components/navigation.vue';
import { useCollection } from '@directus/shared/composables';
import { useSettingsStore, useServerStore } from '@/stores';
import ProjectInfoSidebarDetail from './components/project-info-sidebar-detail.vue';
import { clone } from 'lodash';

export default defineComponent({
	components: { SettingsNavigation, ProjectInfoSidebarDetail },
	setup() {
		const { t } = useI18n();

		const settingsStore = useSettingsStore();
		const serverStore = useServerStore();

		const { fields } = useCollection('directus_settings');

		const initialValues = ref(clone(settingsStore.settings));

		const edits = ref<{ [key: string]: any } | null>(null);

		const noEdits = computed<boolean>(() => edits.value === null || Object.keys(edits.value).length === 0);

		const saving = ref(false);

		return { t, fields, initialValues, edits, noEdits, saving, save };

		async function save() {
			if (edits.value === null) return;
			saving.value = true;
			await settingsStore.updateSettings(edits.value);
			await serverStore.hydrate();
			edits.value = null;
			saving.value = false;
			initialValues.value = clone(settingsStore.settings);
		}
	},
});
</script>

<style lang="scss" scoped>
.settings {
	padding: var(--content-padding);
	padding-bottom: var(--content-padding-bottom);
}

.header-icon {
	--v-button-color-disabled: var(--warning);
	--v-button-background-color-disabled: var(--warning-10);
}
</style>
