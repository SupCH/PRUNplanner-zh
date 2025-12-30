<script setup lang="ts">
	import { PropType, computed } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Composables
	import { useMaterialData } from "@/database/services/useMaterialData";
	const { getMaterialClass } = useMaterialData();

	// Components
	import { Chart } from "highcharts-vue";

	// Util
	import { formatNumber } from "@/util/numbers";

	// Types & Interfaces
	import {
		IEmpireMaterialIO,
		IEmpirePlanListData,
	} from "@/features/empire/empire.types";
	import { Options, SeriesOptionsType } from "highcharts";

	const props = defineProps({
		empireMaterialIO: {
			type: Array as PropType<IEmpireMaterialIO[]>,
			required: true,
		},
		planListData: {
			type: Array as PropType<IEmpirePlanListData[]>,
			required: true,
		},
	});

	// Local State
	const localEmpireMaterialIO = computed(() => props.empireMaterialIO);
	const localPlanListData = computed(() => props.planListData);

	function generatePieChart(
		seriesName: string,
		seriesData: { name: string; y: number; color: string | undefined }[],
		chartType: string = "pie"
	): Options {
		return {
			chart: {
				type: chartType,
				reflow: true,
			},
			title: {
				text: undefined,
			},
			legend: { enabled: false },

			series: [
				{
					name: seriesName,
					data: seriesData,
					dataLabels: {
						enabled: true,
						// eslint-disable-next-line @typescript-eslint/no-explicit-any
						formatter: function (this: any) {
							return `${
								this.key
							}: <span class='font-normal opacity-75'>${formatNumber(
								this.y
							)}</span>`;
						},
					},
				} as unknown as SeriesOptionsType,
			],
		};
	}

	const materialColors: Record<string, string> = {
		"agricultural-products": "#003800",
		alloys: "#7b4c1e",
		chemicals: "#b72e5b",
		"construction-materials": "#185bd3",
		"construction-parts": "#294d6b",
		"construction-prefabs": "#0f1e62",
		"consumable-bundles": "#3e0a11",
		"consumables-basic": "#a62c2a",
		"consumables-luxury": "#680000",
		drones: "#8c3412",
		"electronic-devices": "#561493",
		"electronic-parts": "#5b2eb7",
		"electronic-pieces": "#7752bd",
		"electronic-systems": "#331a4c",
		elements: "#3d2e20",
		"energy-systems": "#153e27",
		fuels: "#548d22",
		gases: "#00696b",
		liquids: "#67a8da",
		"medical-equipment": "#55aa55",
		metals: "#363636",
		minerals: "#997149",
		ores: "#525761",
		plastics: "#791f62",
		"ship-engines": "#992900",
		"ship-kits": "#995400",
		"ship-parts": "#996300",
		"ship-shields": "#bf740a",
		"software-components": "#88792f",
		"software-systems": "#3c3505",
		"software-tools": "#816213",
		textiles: "#525a21",
		"unit-prefabs": "#1d1b1c",
		utility: "#a19488",
	};

	function getMaterialColor(materialTicker: string): string {
		return materialColors[getMaterialClass(materialTicker)];
	}

	const chartProfitablePlans = computed(() => {
		const data = localPlanListData.value.filter((f) => f.profit > 0);

		return generatePieChart(
			t("empire.analysis_charts.profitable_plans"),
			data.map((e) => {
				return {
					name: e.name ?? "",
					y: Math.round(e.profit * 100) / 100,
					color: undefined,
				};
			})
		);
	});

	const chartMaterialProfit = computed(() => {
		const data = localEmpireMaterialIO.value.filter(
			(f) => f.deltaPrice > 0
		);

		return generatePieChart(
			t("empire.analysis_charts.material_profits"),
			data.map((e) => {
				return {
					name: e.ticker ?? "",
					y: Math.round(e.deltaPrice * 100) / 100,
					color: getMaterialColor(e.ticker),
				};
			})
		);
	});

	const chartMaterialCost = computed(() => {
		const data = localEmpireMaterialIO.value.filter(
			(f) => f.deltaPrice < 0
		);

		return generatePieChart(
			t("empire.analysis_charts.material_costs"),
			data.map((e) => {
				return {
					name: e.ticker ?? "",
					y: (Math.round(e.deltaPrice * 100) / 100) * -1,
					color: getMaterialColor(e.ticker),
				};
			})
		);
	});

	const chartNetProduction = computed(() => {
		const data = localEmpireMaterialIO.value.filter((f) => f.delta > 0);

		return generatePieChart(
			t("empire.analysis_charts.net_production"),
			data.map((e) => {
				return {
					name: e.ticker ?? "",
					y: Math.round(e.delta * 100) / 100,
					color: getMaterialColor(e.ticker),
				};
			})
		);
	});

	const chartNetConsumption = computed(() => {
		const data = localEmpireMaterialIO.value.filter((f) => f.delta < 0);

		return generatePieChart(
			t("empire.analysis_charts.net_consumption"),
			data.map((e) => {
				return {
					name: e.ticker ?? "",
					y: (Math.round(e.delta * 100) / 100) * -1,
					color: getMaterialColor(e.ticker),
				};
			})
		);
	});

	const chartExclusiveProduction = computed(() => {
		const data = localEmpireMaterialIO.value.filter(
			(f) => f.output > 0 && f.input === 0
		);

		return generatePieChart(
			t("empire.analysis_charts.exclusive_production"),
			data.map((e) => {
				return {
					name: e.ticker ?? "",
					y: Math.round(e.delta * 100) / 100,
					color: getMaterialColor(e.ticker),
				};
			})
		);
	});

	const chartExclusiveConsumption = computed(() => {
		const data = localEmpireMaterialIO.value.filter(
			(f) => f.output === 0 && f.input > 0
		);

		return generatePieChart(
			t("empire.analysis_charts.exclusive_consumption"),
			data.map((e) => {
				return {
					name: e.ticker ?? "",
					y: (Math.round(e.delta * 100) / 100) * -1,
					color: getMaterialColor(e.ticker),
				};
			})
		);
	});
</script>

<template>
	<div class="border rounded-[3px] border-white/15 p-3">
		<div class="grid grid-cols-1 xl:grid-cols-2 gap-3">
			<div class="col-span-2">
				<h2 class="text-lg font-bold">{{ $t("empire.analysis_charts.profitable_plans") }}</h2>
				<chart
					ref="chartProfitablePlans"
					key="chartProfitablePlans"
					:options="chartProfitablePlans" />
			</div>
			<div>
				<h2 class="text-lg font-bold">{{ $t("empire.analysis_charts.material_profits") }}</h2>

				<chart
					ref="chartMaterialProfit"
					key="chartMaterialProfit"
					:options="chartMaterialProfit" />
			</div>
			<div>
				<h2 class="text-lg font-bold">{{ $t("empire.analysis_charts.material_costs") }}</h2>
				<chart
					ref="chartMaterialCost"
					key="chartMaterialCost"
					:options="chartMaterialCost" />
			</div>
			<div>
				<h2 class="text-lg font-bold">{{ $t("empire.analysis_charts.net_production") }}</h2>
				<chart
					ref="chartNetProduction"
					key="chartNetProduction"
					:options="chartNetProduction" />
			</div>
			<div>
				<h2 class="text-lg font-bold">{{ $t("empire.analysis_charts.net_consumption") }}</h2>
				<chart
					ref="chartNetConsumption"
					key="chartNetConsumption"
					:options="chartNetConsumption" />
			</div>
			<div>
				<h2 class="text-lg font-bold">{{ $t("empire.analysis_charts.exclusive_production") }}</h2>
				<chart
					ref="chartExclusiveProduction"
					key="chartExclusiveProduction"
					:options="chartExclusiveProduction" />
			</div>
			<div>
				<h2 class="text-lg font-bold">{{ $t("empire.analysis_charts.exclusive_consumption") }}</h2>
				<chart
					ref="chartExclusiveConsumption"
					key="chartExclusiveConsumption"
					:options="chartExclusiveConsumption" />
			</div>
		</div>
	</div>
</template>
