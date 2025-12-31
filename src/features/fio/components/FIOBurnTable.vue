<script setup lang="ts">
	import { PropType } from "vue";

	// Composables
	import { usePlanetData } from "@/database/services/usePlanetData";
	import { usePreferences } from "@/features/preferences/usePreferences";
	const { getBurnDisplayClass } = usePreferences();
	const { planetNames, loadPlanetName } = usePlanetData();

	// Components
	import MaterialTile from "@/features/material_tile/components/MaterialTile.vue";
	import XITBurnActionButton from "@/features/xit/components/XITBurnActionButton.vue";

	// Util
	import { formatNumber, formatAmount } from "@/util/numbers";

	// Type & Interfaces
	import {
		IFIOBurnTableElement,
		IFIOBurnTableElementMaterial,
	} from "@/features/fio/useFIOBurn.types";

	// UI
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";

	defineProps({
		burnTable: {
			type: Array as PropType<IFIOBurnTableElement[]>,
			required: true,
		},
	});
</script>

<template>
	<XNDataTable :data="burnTable" striped>
		<XNDataTableColumn title="" type="expand">
			<template #render-cell="{ rowData }">
				{{ rowData.planName }}
			</template>
			<template #render-expand="{ rowData }">
				<XNDataTable :data="rowData.burnMaterials" striped>
					<XNDataTableColumn
						key="ticker"
						:title="$t('plan.tools.supply_cart_details.table.ticker')"
						sorter="default">
						<template #render-cell="data">
							<MaterialTile
								:key="data.rowData.ticker"
								:ticker="data.rowData.ticker" />
						</template>
					</XNDataTableColumn>
					<XNDataTableColumn
						key="input"
						:title="$t('plan.tools.supply_cart_details.table.production').replace($t('plan.sections.production'), $t('plan.sections.input'))"
						sorter="default">
						<template #render-cell="data">
							<span
								:class="
									data.rowData.input <= 0
										? 'text-white/50'
										: ''
								">
								{{ formatNumber(data.rowData.input) }}
							</span>
						</template>
					</XNDataTableColumn>
					<XNDataTableColumn
						key="output"
						:title="$t('plan.sections.output')"
						sorter="default">
						<template #render-cell="data">
							<span
								:class="
									data.rowData.output <= 0
										? 'text-white/50'
										: ''
								">
								{{ formatNumber(data.rowData.output) }}
							</span>
						</template>
					</XNDataTableColumn>
					<XNDataTableColumn
						key="delta"
						:title="$t('empire.material_io_table.delta')"
						sorter="default">
						<template #render-cell="data">
							<span
								:class="
									data.rowData.delta >= 0
										? 'text-positive'
										: 'text-negative'
								">
								{{ formatNumber(data.rowData.delta) }}
							</span>
						</template>
					</XNDataTableColumn>
					<XNDataTableColumn
						key="stock"
						:title="$t('plan.tools.supply_cart_details.table.stock')"
						sorter="default">
						<template #render-cell="data">
							{{ formatAmount(data.rowData.stock) }}
						</template>
					</XNDataTableColumn>
					<XNDataTableColumn
						key="exhaustion"
						:title="$t('nav.items.fio_burn')"
						sorter="default">
						<template #render-cell="data">
							<span
								:class="
									getBurnDisplayClass(data.rowData.exhaustion)
										.value
								"
								class="py-1 px-2">
								{{ formatNumber(data.rowData.exhaustion) }}
							</span>
						</template>
					</XNDataTableColumn>
				</XNDataTable>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="planName" :title="$t('empire.plan_list.plan')">
			<template #title>
				<div class="flex flex-row justify-between">
					<div>{{ $t("empire.plan_list.plan") }}</div>
					<div>{{ $t("nav.items.fio_burn") }}</div>
				</div>
			</template>
			<template #render-cell="{ rowData }">
				<div class="flex flex-row justify-between">
					<div class="my-auto">
						<span class="font-bold">
							{{ rowData.planName }}
						</span>
						<span class="!text-white/50">
							&mdash;
							{{
								planetNames[rowData.planetId] ||
								loadPlanetName(rowData.planetId) ||
								$t("empire.plan_list.loading")
							}}
						</span>
					</div>
					<div class="flex flex-row items-center gap-x-3">
						<div>
							<span
								class="py-1 px-2"
								:class="
									getBurnDisplayClass(rowData.minDays).value
								">
								{{ formatNumber(rowData.minDays) }}
							</span>
						</div>
						<div>
							<XITBurnActionButton
								:drawer-title="`XIT 补给: ${rowData.planName}`"
								:elements="
									rowData.burnMaterials.map(
										(e: IFIOBurnTableElementMaterial) => {
											return {
												ticker: e.ticker,
												stock: e.stock,
												delta: e.delta,
											};
										}
									)
								" />
						</div>
					</div>
				</div>
			</template>
		</XNDataTableColumn>
	</XNDataTable>
</template>
