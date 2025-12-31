<script setup lang="ts">
	import { PropType } from "vue";

	// Util
	import { formatAmount, formatNumber } from "@/util/numbers";

	// Types & Interfaces
	import { IFIOSitesRepairTablePlanetElement } from "@/features/fio/useFIORepair.types";

	// UI
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";

	defineProps({
		repairData: {
			type: Array as PropType<IFIOSitesRepairTablePlanetElement[]>,
			required: true,
		},
	});
</script>

<template>
	<h2 class="text-white/80 font-bold text-lg pb-3">{{ $t("search.results.planet") }}</h2>

	<x-n-data-table :data="repairData" striped>
		<x-n-data-table-column
			key="planetName"
			:title="$t('search.results.planet')"
			sorter="default" />
		<x-n-data-table-column
			key="amountProductionBuildings"
			:title="$t('plan.tools.construction_cart_details.building_label')"
			sorter="default" />
		<x-n-data-table-column
			key="averageCondition"
			:title="'⌀ ' + $t('fio.repair.table.condition')"
			sorter="default">
			<template #render-cell="{ rowData }">
				{{ formatNumber(rowData.averageCondition * 100) }} %
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column
			key="minCondition"
			:title="'Min. ' + $t('fio.repair.table.condition')"
			sorter="default">
			<template #render-cell="{ rowData }">
				{{ formatNumber(rowData.minCondition * 100) }} %
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column
			key="maxLastRepairDays"
			:title="$t('fio.repair.table.last_repair')"
			sorter="default">
			<template #render-cell="{ rowData }">
				{{ formatAmount(rowData.maxLastRepairDays) }}
			</template>
		</x-n-data-table-column>
	</x-n-data-table>
</template>
