<template>
	<header ref="headerEl" class="header-bar" :class="{ collapsed, small }">
		<v-button secondary class="nav-toggle" icon rounded @click="$emit('primary')">
			<v-icon :name="primaryActionIcon" />
		</v-button>

		<div v-if="$slots['title-outer:prepend']" class="title-outer-prepend">
			<slot name="title-outer:prepend" />
		</div>

		<div class="title-container" :class="{ full: !$slots['title-outer:append'] }">
			<div class="headline">
				<slot name="headline" />
			</div>

			<div class="title">
				<slot name="title">
					<slot name="title:prepend" />
					<h1 class="type-title">{{ title }}</h1>
					<slot name="title:append" />
				</slot>
			</div>

			<slot name="title-outer:append" />
		</div>

		<div class="spacer" />

		<slot name="actions:prepend" />

		<header-bar-actions :show-sidebar-toggle="showSidebarToggle" @toggle:sidebar="$emit('toggle:sidebar')">
			<slot name="actions" />
		</header-bar-actions>

		<slot name="actions:append" />
	</header>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, onUnmounted } from 'vue';
import HeaderBarActions from '../header-bar-actions';

export default defineComponent({
	components: { HeaderBarActions },
	props: {
		title: {
			type: String,
			default: null,
		},
		showSidebarToggle: {
			type: Boolean,
			default: false,
		},
		primaryActionIcon: {
			type: String,
			default: 'menu',
		},
		small: {
			type: Boolean,
			default: false,
		},
	},
	emits: ['primary', 'toggle:sidebar'],
	setup() {
		const headerEl = ref<Element>();

		const collapsed = ref(false);

		const observer = new IntersectionObserver(
			([e]) => {
				collapsed.value = e.boundingClientRect.y === -1;
			},
			{ threshold: [1] }
		);

		onMounted(() => {
			observer.observe(headerEl.value as HTMLElement);
		});

		onUnmounted(() => {
			observer.disconnect();
		});

		return { headerEl, collapsed };
	},
});
</script>

<style lang="scss" scoped>
.header-bar {
	position: sticky;
	top: -1px;
	left: 0;
	z-index: 5;
	display: flex;
	align-items: center;
	justify-content: flex-start;
	width: 100%;
	height: 61px;
	margin: 0;
	padding: 0 10px;
	background-color: var(--background-page);
	box-shadow: 0;
	transition: box-shadow var(--medium) var(--transition), margin var(--fast) var(--transition);

	.nav-toggle {
		@media (min-width: 960px) {
			display: none;
		}
	}

	.title-outer-prepend {
		display: none;

		@media (min-width: 960px) {
			display: block;
		}
	}

	.title-container {
		position: relative;
		display: flex;
		align-items: center;
		width: 100%;
		max-width: calc(100% - 12px - 44px - 120px - 12px - 8px);
		height: 100%;
		margin-left: 16px;
		overflow: hidden;

		@media (min-width: 600px) {
			max-width: 70%;
		}

		&.full {
			margin-right: 12px;
			padding-right: 0;

			@media (min-width: 600px) {
				margin-right: 20px;
				padding-right: 20px;
			}
		}

		.headline {
			position: absolute;
			top: 2px;
			left: 0;
			color: var(--foreground-subdued);
			white-space: nowrap;
			opacity: 1;
			transition: opacity var(--fast) var(--transition);

			@media (min-width: 600px) {
				top: -2px;
			}
		}

		.title {
			position: relative;
			display: flex;
			align-items: center;
			overflow: hidden;

			.type-title {
				flex-grow: 1;
				width: 100%;
				overflow: hidden;
				white-space: nowrap;
				text-overflow: ellipsis;
			}
		}
	}

	&.small .title-container .headline {
		opacity: 0;
		pointer-events: none;
	}

	&.collapsed {
		box-shadow: 0 4px 7px -4px rgba(0, 0, 0, 0.2);

		.title-container {
			.headline {
				opacity: 0;
				pointer-events: none;
			}
		}
	}

	.spacer {
		flex-grow: 1;
	}

	.sidebar-toggle {
		flex-shrink: 0;
		margin-left: 8px;

		@media (min-width: 960px) {
			display: none;
		}
	}

	@media (min-width: 600px) {
		padding: 0 32px;

		&:not(.small) {
			margin: 24px 0;
		}
	}
}
</style>
