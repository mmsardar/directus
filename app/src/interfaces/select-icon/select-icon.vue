<script setup lang="ts">
import formatTitle from '@directus/format-title';
import { computed, ref } from 'vue';
import { useI18n } from 'vue-i18n';
import icons from './icons.json';
import { socialIcons } from '@/components/v-icon/social-icons';

withDefaults(
	defineProps<{
		value: string | null;
		disabled?: boolean;
		width?: string;
	}>(),
	{
		width: 'half',
	},
);

const emit = defineEmits(['input']);

const { t } = useI18n();

const searchQuery = ref('');

const mergedIcons = [
	...icons,
	{
		name: 'Social',
		icons: socialIcons,
	},
];

const filteredIcons = computed(() => {
	if (searchQuery.value.length === 0) return mergedIcons;

	return mergedIcons.map((group) => {
		const icons = group.icons.filter((icon) => icon.includes(searchQuery.value.toLowerCase()));

		return {
			name: group.name,
			icons,
		};
	});
});

function setIcon(icon: string | null) {
	searchQuery.value = '';

	emit('input', icon);
}

function onClickInput(e: InputEvent, toggle: () => void) {
	if ((e.target as HTMLInputElement).tagName === 'INPUT') toggle();
}

function onKeydownInput(e: KeyboardEvent, activate: () => void) {
	const systemKeys = e.metaKey || e.altKey || e.ctrlKey || e.shiftKey || e.key === 'Tab';

	if (!e.repeat && !systemKeys && (e.target as HTMLInputElement).tagName === 'INPUT') activate();
}
</script>

<template>
	<v-menu attached :disabled="disabled" no-focus-return>
		<template #activator="{ active, activate, deactivate, toggle }">
			<v-input
				v-model="searchQuery"
				:disabled="disabled"
				:placeholder="value ? formatTitle(value) : t('interfaces.select-icon.search_for_icon')"
				:class="{ 'has-value': value }"
				:nullable="false"
				@click="onClickInput($event, toggle)"
				@keydown="onKeydownInput($event, activate)"
			>
				<template v-if="value" #prepend>
					<v-icon clickable :name="value" :class="{ active: value }" @click="toggle" />
				</template>

				<template #append>
					<div class="item-actions">
						<v-remove
							v-if="value !== null"
							deselect
							@action="
								() => {
									setIcon(null);
									deactivate();
								}
							"
						/>

						<v-icon
							v-else
							clickable
							name="expand_more"
							class="open-indicator"
							:class="{ open: active }"
							@click="toggle"
						/>
					</div>
				</template>
			</v-input>
		</template>

		<div class="content" :class="width">
			<template v-for="(group, index) in filteredIcons" :key="group.name">
				<div v-if="group.icons.length > 0" class="icons">
					<v-icon
						v-for="icon in group.icons"
						:key="icon"
						:name="icon"
						:class="{ active: icon === value }"
						clickable
						@click="setIcon(icon)"
					/>
				</div>
				<v-divider v-if="group.icons.length > 0 && index !== filteredIcons.length - 1" />
			</template>
		</div>
	</v-menu>
</template>

<style lang="scss" scoped>
@use '@/styles/mixins';

.item-actions {
	@include mixins.list-interface-item-actions;
}

.v-input.has-value {
	--v-input-placeholder-color: var(--theme--primary);

	&:focus-within {
		--v-input-placeholder-color: var(--theme--form--field--input--foreground-subdued);
	}
}

.content {
	padding: 8px;

	--v-icon-color-hover: var(--theme--form--field--input--foreground);

	.v-icon.active {
		color: var(--theme--primary);
	}

	.v-divider {
		--v-divider-color: var(--theme--background-normal);

		margin: 0 22px;
	}
}

.icons {
	display: grid;
	grid-gap: 8px;
	grid-template-columns: repeat(auto-fit, 24px);
	justify-content: center;
	padding: 20px 0;
	color: var(--theme--form--field--input--foreground-subdued);
}

.open-indicator {
	transform: scaleY(1);
	transition: transform var(--fast) var(--transition);
}

.open-indicator.open {
	transform: scaleY(-1);
}
</style>
