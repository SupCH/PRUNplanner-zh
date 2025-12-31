<script setup lang="ts">
	import { PropType } from "vue";

	// Util
	import { formatAmount } from "@/util/numbers";
	import { capitalizeString } from "@/util/text";

	// Types & Interfaces
	import { IGraphFlowWorkforceAnalysis } from "@/features/production_chain/productionGraph.types";
	import { WORKFORCECOLORS } from "@/features/production_chain/components/ChainNode.types";

	// UI
	import { PTable } from "@/ui";

	defineProps({
		workforceAnalysis: {
			type: Array as PropType<IGraphFlowWorkforceAnalysis>,
			required: true,
		},
	});
</script>

<template>
	<h3 class="font-bold py-3">{{ $t("tools.production_chain.analysis.workforce_heading") }}</h3>
	<PTable striped>
		<thead>
			<tr>
				<th>{{ $t("tools.production_chain.analysis.workforce_label") }}</th>
				<th class="!text-end">{{ $t("tools.production_chain.analysis.required_label") }}</th>
			</tr>
		</thead>
		<tbody>
			<tr
				v-for="workforce in workforceAnalysis"
				:key="workforce.workforce">
				<td>
					<span
						class="py-1 px-2"
						:style="`background-color: ${
							WORKFORCECOLORS[workforce.workforce]
						};`">
						{{ $t("empire.plan_list.cogc_mapping." + workforce.workforce.toUpperCase()) }}
					</span>
				</td>
				<td class="text-end">
					{{ formatAmount(workforce.value) }}
				</td>
			</tr>
		</tbody>
	</PTable>
</template>
