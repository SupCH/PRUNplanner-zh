<script setup lang="ts">
	// Theme
	import {
		NConfigProvider,
		NModalProvider,
		NDialogProvider,
		darkTheme,
		zhCN,
		dateZhCN,
	} from "naive-ui";
	import { prunplannerTheme } from "@/layout/prunplannerNaiveUI";

	// Components

	import App from "@/App.vue";

	// I18n
	import i18n from "@/locales";
	import { computed } from "vue";

	const currentLocale = computed(() => {
		return i18n.global.locale.value === "zh" ? zhCN : null;
	});

	const currentDateLocale = computed(() => {
		return i18n.global.locale.value === "zh" ? dateZhCN : null;
	});

	// Unhead
	import { useHead } from "@unhead/vue";
	useHead({
		title: "PRUNplanner",
	});
</script>

<template>
	<n-config-provider
		:theme="darkTheme"
		:theme-overrides="prunplannerTheme"
		:locale="currentLocale"
		:date-locale="currentDateLocale"
	>
		<n-modal-provider>
			<n-dialog-provider>
				<App />
			</n-dialog-provider>
		</n-modal-provider>
	</n-config-provider>
</template>
