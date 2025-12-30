<script setup lang="ts">
	import { watch, PropType } from "vue";

	// Composables
	import { usePlanetData } from "@/database/services/usePlanetData";
	const { planetNames, loadPlanetNames } = usePlanetData();

	// Util
	import { formatNumber } from "@/util/numbers";

	// Types & Interfaces
	import { IEmpirePlanListData } from "@/features/empire/empire.types";
	import { PLAN_COGCPROGRAM_TYPE } from "@/stores/planningStore.types";
	import { cogcTextMapping } from "@/features/planning_data/usePlan";

	// UI
	import {
		XNDataTable,
		XNDataTableColumn,
		XNDataTableSummaryRow,
		XNDataTableSummaryCell,
	} from "@skit/x.naive-ui";

	const props = defineProps({
		planListData: {
			type: Array as PropType<IEmpirePlanListData[]>,
			required: true,
		},
	});

	watch(
		() => props.planListData,
		() => loadPlanetNames(props.planListData.map((p) => p.planet)),
		{ immediate: true }
	);
</script>

<template>
	<XNDataTable :data="planListData" striped>
		<XNDataTableColumn key="name" :title="$t('empire.plan_list.plan')" sorter="default">
			<template #render-cell="{ rowData }">
				<div class="text-wrap">
					<router-link
						:to="`/plan/${rowData.planet}/${rowData.uuid}`"
						class="text-link-primary font-bold hover:underline">
						{{ rowData.name }}
					</router-link>
				</div>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="planet" :title="$t('empire.plan_list.planet')" sorter="default">
			<template #render-cell="{ rowData }">
				<div class="test-wrap">
					{{ planetNames[rowData.planet] || $t("empire.plan_list.loading") }}
				</div>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="cogc" :title="$t('empire.plan_list.cogc')" sorter="default">
			<template #render-cell="{ rowData }">
				<div class="text-nowrap">
					{{ $t("empire.plan_list.cogc_mapping." + (rowData.cogc || "---")) }}
				</div>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="permits" :title="$t('empire.plan_list.permits')" sorter="default">
			<template #title>
				<div class="text-nowrap">#</div>
			</template>
			<template #render-cell="{ rowData }">
				<div class="text-center">{{ rowData.permits }}</div>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="profit" :title="$t('empire.plan_list.profit')" sorter="default">
			<template #render-cell="{ rowData }">
				<div class="text-nowrap text-end">
					<span
						:class="
							rowData.profit >= 0
								? 'text-positive'
								: 'text-negative'
						">
						{{ formatNumber(rowData.profit) }}
					</span>
					<span class="pl-1 font-light text-white/50">$</span>
				</div>
			</template>
		</XNDataTableColumn>
		<template #empty>
			<div class="flex flex-col gap-y-3">
				<div class="text-center">{{ $t("empire.plan_list.no_plans") }}</div>
				<div class="text-center">
					{{ $t("empire.plan_list.assign_plans") }}
				</div>
			</div>
		</template>
		<template #summary>
			<XNDataTableSummaryRow>
				<XNDataTableSummaryCell key="name" :col-span="5">
					<template #default>
						<strong class="text-white/80">
							{{ $t("empire.plan_list.total_permits") }}
							{{
								planListData.reduce(
									(sum, elem) => sum + elem.permits,
									0
								)
							}}
						</strong>
					</template>
				</XNDataTableSummaryCell>
			</XNDataTableSummaryRow>
		</template>
	</XNDataTable>
</template>
