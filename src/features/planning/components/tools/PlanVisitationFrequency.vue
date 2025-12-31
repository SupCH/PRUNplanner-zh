<script setup lang="ts">
	import { computed, ComputedRef, PropType, ref, Ref, watch } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Composables
	import { usePlanPreferences } from "@/features/preferences/usePlanPreferences";

	// Util
	import { formatAmount, formatNumber } from "@/util/numbers";

	// Types & Interfaces
	import { PSelectOption } from "@/ui/ui.types";
	import { IMaterialIO } from "@/features/planning/usePlanCalculation.types";

	interface IShippingCalculation {
		shipVolume: number;
		shipWeight: number;
		exportDays: number;
		importDays: number;
		exportLimit: string;
		importLimit: string;
	}

	interface IShippingVariant {
		volume: number;
		weight: number;
	}

	// UI
	import { PSelectMultiple, PTable } from "@/ui";

	const props = defineProps({
		stoAmount: {
			type: Number,
			required: true,
		},
		materialIO: {
			type: Array as PropType<IMaterialIO[]>,
			required: true,
		},
		disabled: {
			type: Boolean,
			required: true,
		},
		planUuid: {
			type: String,
			required: false,
			default: undefined,
		},
	});

	// plan preference patch-in
	const planPrefs = computed<ReturnType<typeof usePlanPreferences> | null>(
		() => {
			return !props.disabled && props.planUuid !== undefined
				? usePlanPreferences(props.planUuid)
				: null;
		}
	);

	function getExclusionOptions(data: IMaterialIO[]): PSelectOption[] {
		return data.map((d) => {
			return {
				label: d.ticker,
				value: d.ticker,
			};
		});
	}

	function calculateDailyData(data: IMaterialIO[]) {
		const dailyWeightImport: number = data.reduce(
			(sum, e) => sum + (e.delta < 0 ? e.totalWeight * -1 : 0),
			0
		);
		const dailyWeightExport: number = data.reduce(
			(sum, e) => sum + (e.delta > 0 ? e.totalWeight : 0),
			0
		);
		const dailyVolumeImport: number = data.reduce(
			(sum, e) => sum + (e.delta < 0 ? e.totalVolume * -1 : 0),
			0
		);
		const dailyVolumeExport: number = data.reduce(
			(sum, e) => sum + (e.delta > 0 ? e.totalVolume : 0),
			0
		);
		const dailyWeightTotal: number = dailyWeightImport + dailyWeightExport;
		const dailyVolumeTotal: number = dailyVolumeImport + dailyVolumeExport;

		return {
			storageFilled: Math.max(
				Math.min(
					totalStorage.value / dailyWeightTotal,
					totalStorage.value / dailyVolumeTotal
				),
				0
			),
			dailyWeightImport: dailyWeightImport,
			dailyWeightExport: dailyWeightExport,
			dailyVolumeImport: dailyVolumeImport,
			dailyVolumeExport: dailyVolumeExport,
			dailyWeight: dailyWeightTotal,
			dailyVolume: dailyVolumeTotal,
		};
	}

	// Local State
	const localStoAmount: Ref<number> = ref(props.stoAmount);
	const localMaterialIO: Ref<IMaterialIO[]> = ref(props.materialIO);

	const refMaterialExclusionOption: Ref<PSelectOption[]> = ref(
		getExclusionOptions(props.materialIO)
	);
	const refMaterialExclusions: Ref<string[]> = ref(
		planPrefs.value === null
			? []
			: planPrefs.value.visitationMaterialExclusions.value
	);

	// Prop Watcher
	watch(
		() => props.stoAmount,
		(newAmount: number) => {
			localStoAmount.value = newAmount;
		}
	);
	watch(
		() => props.materialIO,
		(newIO: IMaterialIO[]) => {
			localMaterialIO.value = newIO;
			refMaterialExclusionOption.value = getExclusionOptions(newIO);
		}
	);

	//
	const totalStorage: ComputedRef<number> = computed(() => {
		return 1500 + 5000 * localStoAmount.value;
	});

	const dailyData = computed(() => {
		const filteredMaterialIO: IMaterialIO[] = localMaterialIO.value.filter(
			(e) => !refMaterialExclusions.value.includes(e.ticker)
		);
		return calculateDailyData(filteredMaterialIO);
	});

	const visitationData = computed(() => {
		const shipVariants: IShippingVariant[] = [
			{ volume: 500, weight: 500 },
			{ volume: 1000, weight: 1000 },
			{ volume: 2000, weight: 2000 },
			{ volume: 3000, weight: 1000 },
			{ volume: 1000, weight: 3000 },
			{ volume: 5000, weight: 5000 },
		];

		const filteredMaterialIO: IMaterialIO[] = localMaterialIO.value.filter(
			(e) => !refMaterialExclusions.value.includes(e.ticker)
		);

		const shippingCalc: IShippingCalculation[] = [];

		const filteredDailyData = calculateDailyData(filteredMaterialIO);

		// calculate per shipVariant
		shipVariants.forEach((ship) => {
			const exportVolumeDays: number =
				ship.volume / filteredDailyData.dailyVolumeExport;
			const exportWeightDays: number =
				ship.weight / filteredDailyData.dailyWeightExport;

			const importVolumeDays: number =
				ship.volume / filteredDailyData.dailyVolumeImport;
			const importWeightDays: number =
				ship.weight / filteredDailyData.dailyWeightImport;

			const exportDays: number =
				exportVolumeDays < exportWeightDays
					? exportVolumeDays
					: exportWeightDays;
			const exportLimit: string =
				exportVolumeDays < exportWeightDays ? "m³" : "t";

			const importDays: number =
				importVolumeDays < importWeightDays
					? importVolumeDays
					: importWeightDays;
			const importLimit: string =
				importVolumeDays < importWeightDays ? "m³" : "t";

			shippingCalc.push({
				shipVolume: ship.volume,
				shipWeight: ship.weight,
				exportDays: exportDays,
				importDays: importDays,
				exportLimit: exportLimit,
				importLimit: importLimit,
			});
		});

		return shippingCalc;
	});
</script>

<template>
	<h2 class="pb-3 text-white/80 font-bold text-lg">{{ $t("plan.tools.visitation_frequency_details.title") }}</h2>

	<div class="grid grid-cols-1 lg:grid-cols-[40%_auto] gap-3">
		<div>
			<h3 class="font-bold text-lg pb-3">{{ $t("plan.tools.visitation_frequency_details.storage_heading") }}</h3>

			<p class="pb-3" v-html="$t('plan.tools.visitation_frequency_details.storage_info', { sto: localStoAmount, total: formatAmount(totalStorage) })">
			</p>

			<PTable striped>
				<thead>
					<tr>
						<th />
						<th class="!text-center">m³</th>
						<th class="!text-center">t</th>
					</tr>
				</thead>
				<tbody class="child:child:text-center">
					<tr>
						<td class="!text-left font-bold">{{ $t("plan.storage.import") }}</td>
						<td>{{ formatNumber(dailyData.dailyVolumeImport) }}</td>
						<td>{{ formatNumber(dailyData.dailyWeightImport) }}</td>
					</tr>
					<tr>
						<td class="!text-left font-bold">{{ $t("plan.storage.export") }}</td>
						<td>{{ formatNumber(dailyData.dailyVolumeExport) }}</td>
						<td>{{ formatNumber(dailyData.dailyWeightExport) }}</td>
					</tr>
					<tr>
						<td class="!text-left font-bold">&#8721;</td>
						<td>{{ formatNumber(dailyData.dailyVolume) }}</td>
						<td>{{ formatNumber(dailyData.dailyWeight) }}</td>
					</tr>
					<tr>
						<td class="!text-left font-bold">{{ $t("plan.tools.visitation_frequency_details.storage_filled") }}</td>
						<td colspan="2" class="font-bold">
							{{ formatNumber(dailyData.storageFilled) }} {{ $t("plan.status_bar.none") === "None" ? "days" : "天" }}
						</td>
					</tr>
				</tbody>
			</PTable>

			<p class="py-3">
				{{ $t("plan.tools.visitation_frequency_details.exclusion_info") }}
			</p>

			<PSelectMultiple
				v-model:value="refMaterialExclusions"
				:disabled="disabled"
				:options="refMaterialExclusionOption"
				multiple
				searchable
				:placeholder="$t('plan.production.material')"
				@update:value="
					(value) => {
						// only keep string values
						const stringsOnly = value.filter((v): v is string => typeof v === 'string');
						if (planPrefs !== null) {
							planPrefs.visitationMaterialExclusions.value =
								stringsOnly;
						}
						refMaterialExclusions = stringsOnly;
					}
				" />
		</div>
		<div>
			<h3 class="font-bold text-lg pb-3">{{ $t("plan.tools.visitation_frequency_details.shipping_heading") }}</h3>

			<PTable striped>
				<thead>
					<tr>
						<th>{{ $t("plan.tools.visitation_frequency_details.table.ship_volume") }}</th>
						<th>{{ $t("plan.tools.visitation_frequency_details.table.ship_weight") }}</th>
						<th class="!text-center">{{ $t("plan.tools.visitation_frequency_details.table.visitation_days") }}</th>
						<th class="!text-center">{{ $t("plan.tools.visitation_frequency_details.table.limit") }}</th>
						<th class="!text-center">{{ $t("plan.tools.visitation_frequency_details.table.visitation_days") }}</th>
						<th class="!text-center">{{ $t("plan.tools.visitation_frequency_details.table.limit") }}</th>
					</tr>
					<tr>
						<th colspan="2" />
						<th colspan="2" class="!text-center">
							{{ $t("plan.tools.visitation_frequency_details.table.export_frequency") }}
						</th>
						<th colspan="2" class="!text-center">
							{{ $t("plan.tools.visitation_frequency_details.table.import_frequency") }}
						</th>
					</tr>
				</thead>
				<tbody>
					<tr
						v-for="(shipData, index) in visitationData"
						:key="index">
						<td>{{ formatAmount(shipData.shipVolume) }}</td>
						<td>{{ formatAmount(shipData.shipWeight) }}</td>
						<td class="text-center">
							{{ formatNumber(shipData.exportDays) }}
						</td>
						<td class="text-center">{{ shipData.exportLimit }}</td>
						<td class="text-center">
							{{ formatNumber(shipData.importDays) }}
						</td>
						<td class="text-center">{{ shipData.importLimit }}</td>
					</tr>
				</tbody>
			</PTable>
		</div>
	</div>
</template>