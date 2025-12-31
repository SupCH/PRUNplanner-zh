<script setup lang="ts">
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();
	import { useVersionCheck } from "@/lib/useVersionCheck";
	const { markUpdated } = useVersionCheck();

	import { trackEvent } from "@/lib/analytics/useAnalytics";

	async function reload(): Promise<void> {
		markUpdated()
			.then(() => {
				trackEvent("version_reload");
			})
			.finally(() => {
				window.location.reload();
			});
	}
</script>

<template>
	<div
		class="absolute z-9999 top-3 right-5 py-2 px-3 bg-prunplanner border border-white/40 shadow-lg cursor-pointer child:text-center text-xs max-w-[300px]"
		@click="reload">
		<div class="uppercase" v-html="$t('loading.version_update.updated')">
		</div>
		<div class="text-black/80">{{ $t("loading.version_update.click_to_load") }}</div>
	</div>
</template>
