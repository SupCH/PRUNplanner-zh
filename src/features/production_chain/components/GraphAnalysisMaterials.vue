<script setup lang="ts">
	import { PropType } from "vue";

	import MaterialTile from "@/features/material_tile/components/MaterialTile.vue";

	// Util
	import { formatNumber } from "@/util/numbers";

	// Components

	// Types & Interfaces
	import { IGraphFlowMaterialAnalysis } from "@/features/production_chain/productionGraph.types";

	// UI
	import { PTable } from "@/ui";

	defineProps({
		materialAnalysis: {
			type: Object as PropType<IGraphFlowMaterialAnalysis>,
			required: true,
		},
	});
</script>

<template>
	<h3 class="font-bold py-3">{{ $t("tools.production_chain.analysis.materials") }}</h3>
	<PTable striped>
		<thead>
			<tr>
				<th>{{ $t("plan.production.material") }}</th>
				<th>{{ $t("plan.workforce.capacity") }}</th>
			</tr>
		</thead>
		<tbody>
			<tr
				v-for="material in materialAnalysis"
				:key="material.materialTicker">
				<td>
					<MaterialTile
						:key="material.materialTicker"
						:ticker="material.materialTicker" />
				</td>
				<td>
					{{ formatNumber(material.amount) }}
				</td>
			</tr>
		</tbody>
	</PTable>
</template>
