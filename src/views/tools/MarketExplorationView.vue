<script setup lang="ts">
	import { onMounted, Ref, ref } from "vue";
	import { useHead } from "@unhead/vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	useHead({
		title: `${t("tools.market_exploration.title")} | PRUNplanner`,
	});

	// Stores
	import { useMaterialData } from "@/database/services/useMaterialData";
	const { materialSelectOptions, preload: preloadMaterials } =
		useMaterialData();

	// Components
	import WrapperGameDataLoader from "@/features/wrapper/components/WrapperGameDataLoader.vue";
	import HelpDrawer from "@/features/help/components/HelpDrawer.vue";

	// Composables
	import { useMarketExplorationChart } from "@/features/market_exploration/useMarketExplorationChart";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Types & Interfaces
	import { PSelectOption } from "@/ui/ui.types";

	// UI
	import { PSelect, PButton, PSpin } from "@/ui";
	import { Chart } from "highcharts-vue";
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";
	import { formatDate } from "@/util/date";
	import { formatAmount } from "@/util/numbers";

	const exchangeOptions: Ref<PSelectOption[]> = ref(
		["AI1", "CI1", "CI2", "IC1", "NC1", "NC2"].map((e) => {
			return { label: e, value: e };
		})
	);

	const materialOptions = ref<PSelectOption[]>([]);

	const selectedExchange: Ref<string> = ref("AI1");
	const selectedMaterial: Ref<string> = ref("RAT");

	const {
		fetchData,
		isLoading: loading,
		hasError: error,
		chartOptions,
		dataChart,
	} = useMarketExplorationChart(selectedExchange, selectedMaterial);

	function fetch(): void {
		trackEvent("marketexploration_explore", {
			exchange: selectedExchange.value,
			materialTicker: selectedMaterial.value,
		});
		fetchData();
	}

	onMounted(async () => {
		await preloadMaterials();
		materialOptions.value = materialSelectOptions.value;
	});
</script>

<template>
	<WrapperGameDataLoader
		:key="`GAMEDATAWRAPPER#MarketExploration`"
		load-materials>
		<div class="min-h-screen flex flex-col">
			<div
				class="px-6 py-3 border-b border-white/10 flex flex-row justify-between gap-x-3">
				<h1 class="text-2xl font-bold">{{ $t("tools.market_exploration.title") }}</h1>
				<div>
					<div class="flex flex-row gap-x-3 child:my-auto">
						<div>{{ $t("tools.market_exploration.exchange_label") }}</div>
						<PSelect
							v-model:value="selectedExchange"
							:options="exchangeOptions"
							class="w-[100px]" />

						<div>{{ $t("tools.market_exploration.material_label") }}</div>
						<PSelect
							v-model:value="selectedMaterial"
							searchable
							:options="materialOptions"
							class="w-[200px]" />
						<PButton :loading="loading" @click="fetch">
							{{ $t("tools.market_exploration.explore_button") }}
						</PButton>
						<HelpDrawer file-name="tools_market_exploration" />
					</div>
				</div>
			</div>
			<div class="px-6 py-3">
				<div v-if="loading" class="text-center">
					<PSpin size="xl" /> <br />
					{{ $t("tools.market_exploration.loading_backend") }}
				</div>
				<div v-else-if="error" class="text-center">
					{{ $t("tools.market_exploration.error_loading") }}
				</div>
				<div v-else-if="!loading && !error && dataChart.length > 0">
					<Chart
						ref="marketchart"
						:constructor-type="'stockChart'"
						class="hc"
						:options="chartOptions" />
					<h2 class="text-xl py-2">{{ $t("tools.market_exploration.data_heading") }}</h2>
					<XNDataTable
						:data="dataChart"
						striped
						:pagination="{ pageSize: 50 }">
						<XNDataTableColumn
							key="Datetime"
							:title="$t('tools.market_exploration.table.date')"
							sorter="default">
							<template #render-cell="{ rowData }">
								{{ formatDate(rowData.Datetime) }}
							</template>
						</XNDataTableColumn>
						<XNDataTableColumn
							key="price_average"
							:title="$t('tools.market_exploration.table.avg_price')"
							sorter="default" />
						<XNDataTableColumn
							key="price_min"
							:title="$t('tools.market_exploration.table.lowest_price')"
							sorter="default" />
						<XNDataTableColumn
							key="price_max"
							:title="$t('tools.market_exploration.table.highest_price')"
							sorter="default" />
						<XNDataTableColumn
							key="volume_max"
							:title="$t('tools.market_exploration.table.traded_volume')"
							sorter="default">
							<template #render-cell="{ rowData }">
								{{ formatAmount(rowData.volume_max) }}
							</template>
						</XNDataTableColumn>
						<XNDataTableColumn
							key="delta_supply_demand"
							:title="$t('tools.market_exploration.table.delta_supply_demand')"
							sorter="default">
							<template #render-cell="{ rowData }">
								<span
									:class="
										rowData.delta_supply_demand >= 0
											? 'text-positive'
											: 'text-negative'
									">
									{{
										formatAmount(
											rowData.delta_supply_demand
										)
									}}
								</span>
							</template>
						</XNDataTableColumn>
					</XNDataTable>
				</div>
				<div v-else class="text-center">
					{{ $t("tools.market_exploration.select_hint") }}
				</div>
			</div>
		</div>
	</WrapperGameDataLoader>
</template>
