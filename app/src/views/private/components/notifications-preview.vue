<script setup lang="ts">
import { storeToRefs } from 'pinia';
import { useI18n } from 'vue-i18n';
import SidebarButton from './sidebar-button.vue';
import NotificationItem from './notification-item.vue';
import { useNotificationsStore } from '@/stores/notifications';
import { useId } from 'vue';

defineProps<{
	sidebarOpen?: boolean;
	modelValue?: boolean;
}>();

defineEmits<{
	(e: 'update:modelValue', value: boolean): void;
}>();

const { t } = useI18n();

const notificationsStore = useNotificationsStore();
const { lastFour } = storeToRefs(notificationsStore);
</script>

<template>
	<div class="notifications-preview">
		<sidebar-button
			v-tooltip.left="!sidebarOpen && t('activity_log')"
			:active="modelValue"
			class="toggle"
			icon="pending_actions"
			@click="$emit('update:modelValue', !modelValue)"
		>
			{{ t('activity_log') }}
		</sidebar-button>

		<transition-expand tag="div">
			<div v-if="modelValue" class="inline">
				<div class="padding-box">
					<router-link class="link" to="/activity" :class="{ 'has-items': lastFour.length > 0 }">
						{{ t('show_all_activity') }}
					</router-link>
					<transition-group tag="div" name="notification" class="transition">
						<notification-item v-for="notification in lastFour" :key="notification.id" v-bind="notification" />
					</transition-group>
				</div>
			</div>
		</transition-expand>
	</div>
</template>

<style lang="scss" scoped>
.notifications-preview {
	position: relative;
}

.link {
	display: block;
	color: var(--theme--foreground-subdued);
	text-align: center;
	text-decoration: none;

	&:hover {
		color: var(--theme--foreground);
	}

	&.has-items {
		margin-bottom: 12px;
	}
}

.transition {
	position: relative;
	width: 100%;
}

.sidebar-button {
	color: var(--theme--sidebar--section--toggle--foreground);
	background-color: var(--theme--sidebar--section--toggle--background);
	--v-icon-color: var(--theme--sidebar--section--toggle--icon--foreground);
}

.inline {
	position: absolute;
	right: 0;
	bottom: 100%;
	width: 100%;
	background-color: var(--theme--background-normal);
	box-shadow: 0px -4px 12px rgb(38 50 56 / 0.1);

	.padding-box {
		position: relative;
		width: 100%;
		padding: 12px;
	}
}

.notification-enter-active,
.notification-leave-active {
	transition: all var(--slow) var(--transition);
}

.notification-leave-active {
	position: absolute;
}

.notification-move {
	transition: all 500ms var(--transition);
}

.notification-enter-from,
.notification-leave-to {
	opacity: 0;
}
</style>
