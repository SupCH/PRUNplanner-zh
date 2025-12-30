<script setup lang="ts">
	import { computed, ComputedRef, PropType, ref, Ref, watch } from "vue";

	// Composables
	import { usePlanetSearchResults } from "../usePlanetSearchResults";

	// Util
	import { formatNumber } from "@/util/numbers";
	import { capitalizeString } from "@/util/text";

	// Components
	import MaterialTile from "@/features/material_tile/components/MaterialTile.vue";
	import PlanetPOPRButton from "@/features/government/components/PlanetPOPRButton.vue";

	// Types & Interfaces
	import { IPlanet } from "@/features/api/gameData.types";
	import { IPlanetSearchResult } from "../usePlanetSearchResults.types";

	// UI
	import { PButton, PTooltip } from "@/ui";
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";
	import { PlusSharp } from "@vicons/material";

	const props = defineProps({
		results: {
			type: Array as PropType<IPlanet[]>,
			required: true,
		},
		searchMaterials: {
			type: Array as PropType<string[]>,
			required: true,
		},
	});

	// Local State
	const localSearchMaterials: ComputedRef<string[]> = computed(
		() => props.searchMaterials
	);
	const localResults: ComputedRef<IPlanet[]> = computed(() => props.results);

	// Table Data
	const tableResults: Ref<IPlanetSearchResult[]> = ref([]);
	const tableSearchMaterials: Ref<string[]> = ref([]);
	const tableCheckDistances: Ref<string | null> = ref(null);

	function prepareTableData(): void {
		tableSearchMaterials.value = [...props.searchMaterials];

		tableResults.value = usePlanetSearchResults(
			localResults.value,
			localSearchMaterials.value
		).results.value;
		tableCheckDistances.value = usePlanetSearchResults(
			localResults.value,
			localSearchMaterials.value
		).hasCheckDistance.value;
	}

	watch(
		() => localResults.value,
		(newResults: IPlanet[]) => {
			if (newResults.length > 0) prepareTableData();
		}
	);
</script>

<template>
	<XNDataTable :data="tableResults" striped :pagination="{ pageSize: 50 }">
		<XNDataTableColumn key="Plan" :title="$t('search.results.plan')" width="50">
			<template #render-cell="{ rowData }">
				<router-link :to="`/plan/${rowData.planetId}`">
					<PButton size="sm">
						<template #icon>
							<PlusSharp />
						</template>
					</PButton>
				</router-link>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="planetName" :title="$t('search.results.planet')" sorter="default">
			<template #render-cell="{ rowData }">
				<span
					v-if="rowData.planetName === rowData.planetId"
					class="!font-bold">
					{{ rowData.planetId }}
				</span>
				<span v-else class="!font-bold">
					{{ rowData.planetName }} ({{ rowData.planetId }})
				</span>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="fertility" :title="$t('search.results.fertility')" sorter="default">
			<template #render-cell="{ rowData }">
				<span v-if="rowData.fertility === 0">&mdash;</span>
				<span v-else>
					{{ formatNumber(rowData.fertility * 100) }} %
				</span>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn
			v-for="sm in tableSearchMaterials"
			:key="`Search#${sm}`"
			:title="sm"
			:sorter="{
				compare: (row1, row2) =>
					// @ts-expect-error naive-ui row typing
					row1.searchResources[sm].dailyExtraction -
					// @ts-expect-error naive-ui row typing
					row2.searchResources[sm].dailyExtraction,
				multiple: 1,
			}">
			<template #render-cell="{ rowData }">
				<div class="child:text-nowrap">
					<MaterialTile
						:key="rowData.searchResources[sm].ticker"
						:ticker="rowData.searchResources[sm].ticker"
						:amount="rowData.searchResources[sm].dailyExtraction"
						:max="rowData.searchResources[sm].maxExtraction" />
				</div>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="additionalResources" :title="$t('search.results.resources')">
			<template #render-cell="{ rowData }">
				<div class="flex flex-row flex-wrap gap-1 child:text-nowrap">
					<MaterialTile
						v-for="am in rowData.additionalResources"
						:key="am.ticker"
						:ticker="am.ticker"
						:amount="am.dailyExtraction"
						:max="am.maxExtraction" />
				</div>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="popr" title="POPR">
			<template #render-cell="{ rowData }">
				<PlanetPOPRButton
					:planet-natural-id="rowData.planetId"
					button-size="sm" />
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn
			key="cogcProgram"
			title="COGC"
			sorter="default">
			<template #render-cell="{ rowData }">
				{{
					capitalizeString(rowData.cogcProgram)
						.replace("Advertising ", "")
						.replace("Workforce ", "")
				}}
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="environment" :title="$t('search.results.environment')">
			<template #render-cell="{ rowData }">
				<div class="flex flex-row flex-wrap gap-1">
					<PTooltip v-if="rowData.environmentSurface.length !== 0">
						<template #trigger>
							<div>
								<MaterialTile
									:key="rowData.environmentSurface[0]"
									:ticker="rowData.environmentSurface[0]" />
							</div>
						</template>
						{{ $t("search.results.env_tips.surface") }}
					</PTooltip>
					<PTooltip v-if="rowData.environmentGravity.length !== 0">
						<template #trigger>
							<div>
								<MaterialTile
									:key="rowData.environmentGravity[0]"
									:ticker="rowData.environmentGravity[0]" />
							</div>
						</template>
						{{ $t("search.results.env_tips.gravity") }}
					</PTooltip>
					<PTooltip
						v-if="rowData.environmentTemperature.length !== 0">
						<template #trigger>
							<div>
								<MaterialTile
									:key="rowData.environmentTemperature[0]"
									:ticker="
										rowData.environmentTemperature[0]
									" />
							</div>
						</template>
						{{ $t("search.results.env_tips.temperature") }}
					</PTooltip>
					<PTooltip v-if="rowData.environmentPressure.length !== 0">
						<template #trigger>
							<div>
								<MaterialTile
									:key="rowData.environmentPressure[0]"
									:ticker="rowData.environmentPressure[0]" />
							</div>
						</template>
						{{ $t("search.results.env_tips.pressure") }}
					</PTooltip>
				</div>
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn key="infrastructures" :title="$t('search.results.infrastructure')">
			<template #render-cell="{ rowData }">
				{{ rowData.infrastructures.join(", ") }}
			</template>
		</XNDataTableColumn>
		<XNDataTableColumn
			v-if="tableCheckDistances !== null"
			key="checkDistance"
			:title="`Distance ${tableCheckDistances}`"
			sorter="default" />
		<XNDataTableColumn key="distanceAI1" title="AI1" sorter="default" />
		<XNDataTableColumn key="distanceCI1" title="CI1" sorter="default" />
		<XNDataTableColumn key="distanceIC1" title="IC1" sorter="default" />
		<XNDataTableColumn key="distanceNC1" title="NC1" sorter="default" />
	</XNDataTable>
</template>
