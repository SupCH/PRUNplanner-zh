<script setup lang="ts">
	import {
		ComputedRef,
		PropType,
		Ref,
		computed,
		onMounted,
		ref,
		watch,
	} from "vue";

	// Composables
	import { useROIOverview } from "@/features/roi_overview/useROIOverview";

	// Util
	import { formatNumber } from "@/util/numbers";
	import { capitalizeString } from "@/util/text";

	// Components
	import MaterialTile from "@/features/material_tile/components/MaterialTile.vue";
	import COGMButton from "@/features/roi_overview/components/COGMButton.vue";
	import ROIOverviewTableFilters from "@/features/roi_overview/components/ROIOverviewTableFilters.vue";

	// Types & Interfaces
	import { IPlan } from "@/stores/planningStore.types";
	import { IROIResult } from "@/features/roi_overview/useROIOverview.types";
	import { PSelectOption } from "@/ui/ui.types";

	// UI
	import { PProgressBar } from "@/ui";
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";

	const props = defineProps({
		planDefinition: {
			type: Object as PropType<IPlan>,
			required: true,
		},
		cxUuid: {
			type: String,
			required: false,
			default: undefined,
		},
	});

	const definition: ComputedRef<IPlan> = computed(() => props.planDefinition);
	const cx: ComputedRef<string | undefined> = computed(() => props.cxUuid);

	const result: Ref<IROIResult[]> = ref([]);
	const isCalculating: Ref<boolean> = ref(true);

	const filterBuilding: Ref<string | null> = ref(null);
	const filterCOGC: Ref<string | null> = ref(null);
	const filterInputMaterial: Ref<string | null> = ref(null);
	const filterOutputMaterial: Ref<string | null> = ref(null);
	const filterPostiveROI: Ref<boolean> = ref(false);

	const { calculate, formatOptimal, progressCurrent, progressTotal } =
		await useROIOverview(definition, cx);

	const filteredResult: ComputedRef<IROIResult[]> = computed(() => {
		let filtered = result.value;

		if (filterBuilding.value !== null)
			filtered = filtered.filter(
				(f) => f.buildingTicker === filterBuilding.value
			);

		if (filterCOGC.value !== null)
			filtered = filtered.filter((f) => f.cogc === filterCOGC.value);

		if (filterInputMaterial.value !== null)
			filtered = filtered.filter((f) =>
				f.recipeInputs
					.map((e) => e.Ticker)
					.includes(filterInputMaterial.value!)
			);

		if (filterOutputMaterial.value !== null)
			filtered = filtered.filter((f) =>
				f.recipeOutputs
					.map((e) => e.Ticker)
					.includes(filterOutputMaterial.value!)
			);

		if (filterPostiveROI.value)
			filtered = filtered.filter((f) => f.planROI >= 0);

		return filtered;
	});

	const filterOptionBuilding: ComputedRef<PSelectOption[]> = computed(() =>
		Array.from(new Set(result.value.map((e) => e.buildingTicker)))
			.sort((a, b) => (a > b ? 1 : -1))
			.map((e) => ({ label: e, value: e }))
	);

	const filterOptionCOGC: ComputedRef<PSelectOption[]> = computed(() =>
		Array.from(new Set(result.value.map((e) => e.cogc)))
			.sort()
			.map((e) => ({ label: capitalizeString(e), value: e }))
	);

	const filterOptionInputMaterial: ComputedRef<PSelectOption[]> = computed(
		() =>
			Array.from(
				new Set(
					result.value
						.map((e) => e.recipeInputs.map((e) => e.Ticker))
						.flat()
				)
			)
				.sort()
				.map((e) => ({ label: e, value: e }))
	);

	const filterOptionOutputMaterial: ComputedRef<PSelectOption[]> = computed(
		() =>
			Array.from(
				new Set(
					result.value
						.map((e) => e.recipeOutputs.map((e) => e.Ticker))
						.flat()
				)
			)
				.sort()
				.map((e) => ({ label: e, value: e }))
	);

	async function get() {
		isCalculating.value = true;
		result.value = [];

		calculate()
			.then((d) => (result.value = d))
			.finally(() => (isCalculating.value = false));
	}

	// watch for cx change, must retrigger calculation
	watch(
		() => props.cxUuid,
		async () => {
			get();
		}
	);

	// trigger initial calculation on load
	onMounted(async () => {
		get();
	});
</script>

<template>
	<div v-if="isCalculating" class="w-full flex justify-center">
		<div class="text-center w-[400px] py-3">
			<PProgressBar :step="progressCurrent" :total="progressTotal" />
			<div class="pt-3 text-xs text-white/60">
				{{ $t("tools.recipe_roi.calculating") }}
			</div>
		</div>
	</div>
	<div v-else>
		<ROIOverviewTableFilters
			v-model:filter-building="filterBuilding"
			v-model:filter-cogc="filterCOGC"
			v-model:filter-output-material="filterOutputMaterial"
			v-model:filter-input-material="filterInputMaterial"
			v-model:filter-positive-r-o-i="filterPostiveROI"
			:building-options="filterOptionBuilding"
			:cogc-options="filterOptionCOGC"
			:output-material-options="filterOptionOutputMaterial"
			:input-material-options="filterOptionInputMaterial" />
		<XNDataTable
			:data="filteredResult"
			striped
			:pagination="{ pageSize: 50 }">
			<XNDataTableColumn
				key="buildingTicker"
				:title="$t('tools.recipe_roi.table.building')"
				sorter="default">
				<template #render-cell="{ rowData }">
					<span class="font-bold">{{ rowData.buildingTicker }}</span>
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn key="optimalSetup" :title="$t('tools.recipe_roi.table.optimal_setup')">
				<template #render-cell="{ rowData }">
					{{ formatOptimal(rowData.optimalSetup) }}
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn key="recipeOutputs" :title="$t('tools.recipe_roi.table.recipe_output')">
				<template #render-cell="{ rowData }">
					<div class="flex flex-row flex-wrap gap-1">
						<MaterialTile
							v-for="output in rowData.recipeOutputs"
							:key="`${rowData.buildingTicker}#output#${output.Ticker}`"
							:ticker="output.Ticker"
							:amount="output.Amount" />
					</div>
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn key="recipeInputs" :title="$t('tools.recipe_roi.table.recipe_input')">
				<template #render-cell="{ rowData }">
					<div class="flex flex-row flex-wrap gap-1">
						<MaterialTile
							v-for="output in rowData.recipeInputs"
							:key="`${rowData.buildingTicker}#input#${output.Ticker}`"
							:ticker="output.Ticker"
							:amount="output.Amount" />
					</div>
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn key="cogc" :title="$t('tools.recipe_roi.table.cogc')">
				<template #render-cell="{ rowData }">
					<span class="text-nowrap">
						{{ $t("plan.experts.mapping." + (rowData.cogc || "NONE").toUpperCase()) }}
					</span>
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn key="cogm" :title="$t('tools.recipe_roi.table.cogm')">
				<template #render-cell="{ rowData }">
					<COGMButton :cogm-data="rowData.cogm" />
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn
				key="planCost"
				:title="$t('tools.recipe_roi.table.plan_cost')"
				sorter="default">
				<template #title>
					<div class="text-end">{{ $t("tools.recipe_roi.table.plan_cost") }}</div>
				</template>
				<template #render-cell="{ rowData }">
					<div class="text-end text-nowrap">
						{{ formatNumber(rowData.planCost) }}
						<span class="pl-1 font-light text-white/50"> $ </span>
					</div>
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn
				key="dailyProfit"
				:title="$t('tools.recipe_roi.table.daily_profit')"
				sorter="default">
				<template #title>
					<div class="text-end">{{ $t("tools.recipe_roi.table.daily_profit") }}</div>
				</template>
				<template #render-cell="{ rowData }">
					<div
						class="text-end text-nowrap"
						:class="
							rowData.dailyProfit > 0
								? 'text-positive'
								: 'text-negative'
						">
						{{ formatNumber(rowData.dailyProfit) }}
						<span class="pl-1 font-light text-white/50"> $ </span>
					</div>
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn
				key="planProfitArea"
				:title="$t('tools.recipe_roi.table.profit_area')"
				sorter="default">
				<template #title>
					<div class="text-end">{{ $t("tools.recipe_roi.table.profit_area") }}</div>
				</template>
				<template #render-cell="{ rowData }">
					<div
						class="text-end"
						:class="
							rowData.planProfitArea > 0
								? 'text-positive'
								: 'text-negative'
						">
						{{ formatNumber(rowData.planProfitArea) }}
						<span class="pl-1 font-light text-white/50"> $ </span>
					</div>
				</template>
			</XNDataTableColumn>
			<XNDataTableColumn key="planROI" :title="$t('tools.recipe_roi.table.plan_roi')" sorter="default">
				<template #title>
					<div class="text-end">{{ $t("tools.recipe_roi.table.plan_roi") }}</div>
				</template>
				<template #render-cell="{ rowData }">
					<div
						class="text-end text-nowrap"
						:class="
							rowData.planROI > 0
								? 'text-positive'
								: 'text-negative'
						">
						{{ formatNumber(rowData.planROI) }}
						<span class="pl-1 font-light text-white/50"> d </span>
					</div>
				</template>
			</XNDataTableColumn>
		</XNDataTable>
	</div>
</template>
