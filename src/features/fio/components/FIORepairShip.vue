<script setup lang="ts">
	import { PropType } from "vue";

	// Util
	import { formatNumber } from "@/util/numbers";

	import MaterialTile from "@/features/material_tile/components/MaterialTile.vue";

	// Types & Interfaces
	import { IFIOSitesRepairTableShipElement } from "@/features/fio/useFIORepair.types";

	// UI
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";

	defineProps({
		repairData: {
			type: Array as PropType<IFIOSitesRepairTableShipElement[]>,
			required: true,
		},
	});
</script>

<template>
	<h2 class="text-white/80 font-bold text-lg pb-3">{{ $t("nav.items.help") === "Help" ? "Ships" : "飞船" }}</h2>

	<x-n-data-table :data="repairData" striped>
		<x-n-data-table-column
			key="shipRegistration"
			:title="$t('search.basic.id_label')"
			sorter="default" />
		<x-n-data-table-column key="shipName" :title="$t('plan.configuration.name_label')" sorter="default" />
		<x-n-data-table-column
			key="condition"
			:title="$t('fio.repair.table.condition')"
			sorter="default">
			<template #render-cell="{ rowData }">
				{{ formatNumber(rowData.condition * 100) }} %
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column
			key="repairMaterials"
			:title="$t('fio.repair.table.repair_cost')"
			sorter="default">
			<template #render-cell="{ rowData }">
				<div class="flex flex-row flex-wrap gap-1 child:text-nowrap">
					<MaterialTile
						v-for="material in rowData.repairMaterials"
						:key="material.ticker"
						:ticker="material.ticker"
						:amount="material.amount" />
				</div>
			</template>
		</x-n-data-table-column>
	</x-n-data-table>
</template>
