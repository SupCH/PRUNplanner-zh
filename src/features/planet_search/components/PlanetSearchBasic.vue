<script setup lang="ts">
	import { computed, ComputedRef, ref, Ref } from "vue";

	// API
	import { useQuery } from "@/lib/query_cache/useQuery";

	// Composables
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Types & Interfaces
	import { IPlanet } from "@/features/api/gameData.types";

	// UI
	import { PForm, PFormItem, PInput, PButton } from "@/ui";
	import { SearchSharp } from "@vicons/material";

	const refSearchId: Ref<string | null> = ref(null);

	const emit = defineEmits<{
		(e: "update:results", value: IPlanet[]): void;
	}>();

	const isLoading: Ref<boolean> = ref(false);
	const canSearch: ComputedRef<boolean> = computed(
		() => refSearchId.value !== null && refSearchId.value.length >= 3
	);

	async function doSearch() {
		if (canSearch.value) {
			isLoading.value = true;

			trackEvent("planet_search_basic", { searchId: refSearchId.value! });

			try {
				await useQuery("GetPlanetSearchSingle", {
					searchId: refSearchId.value!,
				})
					.execute()
					.then((data: IPlanet[]) => emit("update:results", data))
					.finally(() => (isLoading.value = false));
			} catch {
				emit("update:results", []);
			}
		}
	}
</script>

<template>
	<div class="flex flex-row flex-wrap gap-3 justify-between">
		<h2 class="text-lg font-bold my-auto">{{ $t("search.basic.heading") }}</h2>
		<PButton :loading="isLoading" :disabled="!canSearch" @click="doSearch">
			<template #icon><SearchSharp /></template>
			{{ $t("search.basic.search_button") }}
		</PButton>
	</div>

	<div class="py-3 text-white/60">
		{{ $t("search.basic.search_info") }}
	</div>

	<PForm>
		<PFormItem :label="$t('search.basic.id_label')">
			<PInput v-model:value="refSearchId" class="w-full" />
		</PFormItem>
	</PForm>
</template>
