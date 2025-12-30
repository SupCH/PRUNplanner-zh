<script setup lang="ts">
	import { ref, Ref } from "vue";
	import { useHead } from "@unhead/vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	useHead({
		title: `${t("search.title")} | PRUNplanner`,
	});

	// Components
	import WrapperGameDataLoader from "@/features/wrapper/components/WrapperGameDataLoader.vue";
	import HelpDrawer from "@/features/help/components/HelpDrawer.vue";
	import PlanetSearchBasic from "@/features/planet_search/components/PlanetSearchBasic.vue";
	import PlanetSearchAdvanced from "@/features/planet_search/components/PlanetSearchAdvanced.vue";
	import PlanetSearchResults from "@/features/planet_search/components/PlanetSearchResults.vue";

	import { IPlanet } from "@/features/api/gameData.types";

	const refResults: Ref<IPlanet[]> = ref([]);
	const refSearchMaterials: Ref<string[]> = ref([]);
</script>

<template>
	<WrapperGameDataLoader load-materials load-exchanges>
		<div class="min-h-screen flex flex-col">
			<div
				class="px-6 py-3 border-b border-white/10 flex flex-row justify-between">
				<h1 class="text-2xl font-bold my-auto">{{ $t("search.title") }}</h1>
				<HelpDrawer file-name="planet_search" />
			</div>

			<div class="border-b border-white/10">
				<div
					class="grid grid-cols-1 lg:grid-cols-[20%_auto] gap-3 divide-x divide-white/10">
					<div class="px-6 py-3">
						<PlanetSearchBasic
							@update:results="
								(value) => {
									refResults = value;
									refSearchMaterials = [];
								}
							" />
					</div>
					<div class="px-3 py-3">
						<PlanetSearchAdvanced
							@update:results="(value) => (refResults = value)"
							@update:materials="
								(value) => (refSearchMaterials = value)
							" />
					</div>
				</div>
			</div>
			<div class="child:px-6 child:py-3">
				<div>
					<PlanetSearchResults
						:results="refResults"
						:search-materials="refSearchMaterials" />
				</div>
			</div>
		</div>
	</WrapperGameDataLoader>
</template>
