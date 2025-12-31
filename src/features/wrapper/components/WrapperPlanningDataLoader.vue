<script setup lang="ts">
	import { useI18n } from "vue-i18n";
	const { t, te } = useI18n();
	// Composables
	import { usePlanningDataLoader } from "@/features/wrapper/usePlanningDataLoader";

	// Types & Interfaces
	import {
		PlanningDataLoaderEmits,
		PlanningDataLoaderProps,
	} from "@/features/wrapper/planningDataLoader.types";

	// Components
	import RenderingProgress from "@/layout/components/RenderingProgress.vue";

	// UI
	import { PSpin, PIcon } from "@/ui";
	import { CheckSharp, ClearSharp } from "@vicons/material";

	const props: PlanningDataLoaderProps =
		defineProps<PlanningDataLoaderProps>();
	const emit = defineEmits<PlanningDataLoaderEmits>();

	const { done, allLoaded, hasError, loadingSteps, results } =
		usePlanningDataLoader(props, emit);
</script>

<template>
	<template v-if="!done && !allLoaded">
		<div
			class="relative w-full h-full bg-center bg-repeat"
			:class="
				!hasError
					? 'bg-[url(/images/bg_striped_prunplanner.png)]'
					: 'bg-[url(/images/bg_striped_error.png)]'
			">
			<div class="absolute inset-0 flex items-center justify-center">
				<div
					class="min-w-[300px] max-w-[500px] bg-black p-8 rounded shadow-lg text-center flex flex-col gap-y-3">
					<h1 class="text-2xl font-bold font-mono mb-3">
						{{ $t("loading.title") }}
					</h1>
					<div
						v-for="e in loadingSteps"
						:key="e.name"
						class="flex flex-row align-middle gap-x-3">
						<div class="mr-5 w-[30px]">
							<div v-if="e.loading" class="my-1">
								<PSpin />
							</div>
							<PIcon v-else :size="20">
								<CheckSharp v-if="!e.error" />
								<ClearSharp v-else />
							</PIcon>
						</div>
						<div>
							{{ te("loading.steps." + e.name) 
								? $t("loading.steps." + e.name) 
								: e.name.includes("Planet") 
									? e.name.replace("Planet", $t("loading.steps.Planet")).replace("Data", $t("loading.steps.Material Data").replace($t("loading.steps.Materials"), "").trim()).replace(/'/g, "")
									: e.name 
							}}
						</div>
					</div>
				</div>
			</div>
		</div>
	</template>
	<Suspense v-else>
		<slot
			:complete="allLoaded"
			:disabled="results.disabled"
			:shared-plan="results.sharedPlan"
			:empire-list="results.empireList"
			:empire-planet-list="results.empirePlanetList.value"
			:planet-data="results.planetData"
			:plan-data="results.planData"
			:plan-list="results.planList"
			:shared-data="results.sharedData"
			:plan-definition="results.planDefinition" />
		<template #fallback>
			<RenderingProgress />
		</template>
	</Suspense>
</template>
