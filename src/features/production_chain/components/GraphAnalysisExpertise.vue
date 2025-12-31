<script setup lang="ts">
	import { PropType } from "vue";

	// Util
	import { formatAmount } from "@/util/numbers";
	import { capitalizeString } from "@/util/text";

	// Types & Interfaces
	import { IGraphFlowExpertiseAnalyis } from "@/features/production_chain//productionGraph.types";
	import { EXPERTISECOLORS } from "@/features/production_chain/components/ChainNode.types";
	import { BUILDING_EXPERTISE_TYPE } from "@/features/api/gameData.types";

	// UI
	import { PTable } from "@/ui";

	defineProps({
		expertiseAnalysis: {
			type: Object as PropType<IGraphFlowExpertiseAnalyis>,
			required: true,
		},
	});
</script>

<template>
	<h3 class="font-bold pb-3">{{ $t("tools.production_chain.analysis.expertise_heading") }}</h3>

	<PTable striped>
		<thead>
			<tr>
				<th>{{ $t("tools.production_chain.analysis.expertise_label") }}</th>
				<th class="!text-end">{{ $t("plan.production.material") }}</th>
			</tr>
		</thead>
		<tbody>
			<tr
				v-for="[expertise, value] in Object.entries(expertiseAnalysis)"
				:key="expertise">
				<td>
					<span
						class="py-1 px-2"
						:style="`background-color:
											${EXPERTISECOLORS[expertise as
											BUILDING_EXPERTISE_TYPE]};`">
						{{ $t("plan.experts.mapping." + expertise.toUpperCase()) }}
					</span>
				</td>
				<td class="text-end">
					{{ formatAmount(value) }}
				</td>
			</tr>
		</tbody>
	</PTable>
</template>
