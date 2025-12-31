<script setup lang="ts">
	import { computed } from "vue";

	// Components
	import MaterialTile from "@/features/material_tile/components/MaterialTile.vue";

	// Util
	import { formatNumber } from "@/util/numbers";

	// Types & Interfaces
	import { IMaterialIO } from "@/features/planning/usePlanCalculation.types";

	// UI
	import { PTable } from "@/ui";

	const { materials } = defineProps<{
		materials: IMaterialIO[];
	}>();

	const totalData = computed(() => {
		return materials.reduce(
			(acc, current) => {
				acc.cost += current.price * -1;
				acc.weight += current.totalWeight * -1;
				acc.volume += current.totalVolume * -1;
				return acc;
			},
			{ cost: 0, weight: 0, volume: 0 }
		);
	});
</script>

<template>
	<PTable striped>
		<thead>
			<tr>
				<th>{{ $t("plan.tools.hq_calculator.table.material") }}</th>
				<th>{{ $t("plan.tools.hq_calculator.table.amount") }}</th>
				<th class="!text-end">{{ $t("plan.tools.hq_calculator.table.unit_cost") }}</th>
			</tr>
		</thead>
		<tbody>
			<tr
				v-for="material in materials"
				:key="`RepairMaterial#${material.ticker}`">
				<td>
					<MaterialTile
						:key="`RepairMaterial#Tile#${material.ticker}`"
						:ticker="material.ticker" />
				</td>
				<td>
					{{ material.input }}
				</td>
				<td class="text-end">
					{{ formatNumber(-1 * material.price) }}
					<span class="pl-1 font-light text-white/50"> $ </span>
				</td>
			</tr>
		</tbody>
		<tfoot>
			<tr>
				<td colspan="3" class="!border-t">
					<div
						class="grid grid-cols-2 gap-1 child:even:text-end child:not-even:font-bold">
						<div>{{ $t("plan.tools.hq_calculator.total_cost") }}</div>
						<div>
							{{ formatNumber(totalData.cost) }}
							<span class="pl-1 font-light text-white/50">
								$
							</span>
						</div>
						<div>{{ $t("plan.tools.hq_calculator.total_volume") }}</div>
						<div>
							{{ formatNumber(totalData.volume) }}
							<span class="pl-1 font-light text-white/50">
								m³
							</span>
						</div>
						<div>{{ $t("plan.tools.hq_calculator.total_weight") }}</div>
						<div>
							{{ formatNumber(totalData.weight) }}
							<span class="pl-1 font-light text-white/50">
								t
							</span>
						</div>
					</div>
				</td>
			</tr>
		</tfoot>
	</PTable>
</template>
