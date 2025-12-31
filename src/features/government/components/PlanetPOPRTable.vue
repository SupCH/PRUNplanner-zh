<script setup lang="ts">
	import { PropType } from "vue";

	// Util
	import { formatAmount, formatNumber } from "@/util/numbers";

	// Types & Interfaces
	import { IPopulationReport } from "@/features/api/gameData.types";
	import { IWorkforceRecord } from "@/features/planning/usePlanCalculation.types";

	// UI
	import { PTable, PIcon } from "@/ui";
	import { CheckSharp, ClearSharp } from "@vicons/material";

	defineProps({
		planetNaturalId: {
			type: String,
			required: true,
		},
		poprData: {
			type: Object as PropType<IPopulationReport>,
			required: false,
			default: undefined,
		},
		workforceData: {
			type: Object as PropType<IWorkforceRecord>,
			required: false,
			default: undefined,
		},
	});

	function assessPlanWorkforce(
		free: number,
		unfilled: number,
		plan: number
	): boolean {
		if (plan == 0) {
			return true;
		}

		if (free - unfilled >= plan) {
			return true;
		} else {
			return false;
		}
	}
</script>

<template>
	<div v-if="!poprData" class="text-center">
		{{ $t("plan.tools.popr_details.no_data", { planet: planetNaturalId }) }}
	</div>

	<PTable striped>
		<thead>
			<tr>
				<th></th>
				<th>{{ $t("empire.plan_list.cogc_mapping.PIONEERS") }}</th>
				<th>{{ $t("empire.plan_list.cogc_mapping.SETTLERS") }}</th>
				<th>{{ $t("empire.plan_list.cogc_mapping.TECHNICIANS") }}</th>
				<th>{{ $t("empire.plan_list.cogc_mapping.ENGINEERS") }}</th>
				<th>{{ $t("empire.plan_list.cogc_mapping.SCIENTISTS") }}</th>
			</tr>
		</thead>
		<tbody>
			<tr>
				<td class="font-bold">{{ $t("plan.tools.popr_details.table.total_population") }}</td>
				<td>{{ formatAmount(poprData.NextPopulationPioneer) }}</td>
				<td>{{ formatAmount(poprData.NextPopulationSettler) }}</td>
				<td>{{ formatAmount(poprData.NextPopulationTechnician) }}</td>
				<td>{{ formatAmount(poprData.NextPopulationEngineer) }}</td>
				<td>{{ formatAmount(poprData.NextPopulationScientist) }}</td>
			</tr>
			<tr>
				<td class="font-bold">{{ $t("plan.tools.popr_details.table.free_population") }}</td>
				<td>{{ formatAmount(poprData.FreePioneer) }}</td>
				<td>{{ formatAmount(poprData.FreeSettler) }}</td>
				<td>{{ formatAmount(poprData.FreeTechnician) }}</td>
				<td>{{ formatAmount(poprData.FreeEngineer) }}</td>
				<td>{{ formatAmount(poprData.FreeScientist) }}</td>
			</tr>
			<tr>
				<td class="font-bold">{{ $t("plan.tools.popr_details.table.free_population_percent") }}</td>
				<td>
					{{ formatNumber(poprData.UnemploymentRatePioneer * 100) }}
				</td>
				<td>
					{{ formatNumber(poprData.UnemploymentRateSettler * 100) }}
				</td>
				<td>
					{{
						formatNumber(poprData.UnemploymentRateTechnician * 100)
					}}
				</td>
				<td>
					{{ formatNumber(poprData.UnemploymentRateEngineer * 100) }}
				</td>
				<td>
					{{ formatNumber(poprData.UnemploymentRateScientist * 100) }}
				</td>
			</tr>
			<tr>
				<td class="font-bold">{{ $t("plan.tools.popr_details.table.unfilled_jobs") }}</td>
				<td>{{ formatAmount(poprData.OpenJobsPioneer) }}</td>
				<td>{{ formatAmount(poprData.OpenJobsSettler) }}</td>
				<td>{{ formatAmount(poprData.OpenJobsTechnician) }}</td>
				<td>{{ formatAmount(poprData.OpenJobsEngineer) }}</td>
				<td>{{ formatAmount(poprData.OpenJobsScientist) }}</td>
			</tr>
			<template v-if="workforceData">
				<tr class="child:!border-t-2 child:!border-t-white/20">
					<td class="font-bold">{{ $t("plan.tools.popr_details.table.plan_need") }}</td>
					<td>{{ formatAmount(workforceData.pioneer.required) }}</td>
					<td>{{ formatAmount(workforceData.settler.required) }}</td>
					<td>
						{{ formatAmount(workforceData.technician.required) }}
					</td>
					<td>{{ formatAmount(workforceData.engineer.required) }}</td>
					<td>
						{{ formatAmount(workforceData.scientist.required) }}
					</td>
				</tr>
				<tr>
					<td class="font-bold">{{ $t("plan.tools.popr_details.table.plan_check") }}</td>
					<td>
						<PIcon>
							<component
								:is="
									assessPlanWorkforce(
										poprData.FreePioneer,
										poprData.OpenJobsPioneer,
										workforceData.pioneer.required
									)
										? CheckSharp
										: ClearSharp
								"
								:class="
									assessPlanWorkforce(
										poprData.FreePioneer,
										poprData.OpenJobsPioneer,
										workforceData.pioneer.required
									)
										? 'text-positive'
										: 'text-negative'
								" />
						</PIcon>
					</td>
					<td>
						<PIcon>
							<component
								:is="
									assessPlanWorkforce(
										poprData.FreeSettler,
										poprData.OpenJobsSettler,
										workforceData.settler.required
									)
										? CheckSharp
										: ClearSharp
								"
								:class="
									assessPlanWorkforce(
										poprData.FreeSettler,
										poprData.OpenJobsSettler,
										workforceData.settler.required
									)
										? 'text-positive'
										: 'text-negative'
								" />
						</PIcon>
					</td>
					<td>
						<PIcon>
							<component
								:is="
									assessPlanWorkforce(
										poprData.FreeTechnician,
										poprData.OpenJobsTechnician,
										workforceData.technician.required
									)
										? CheckSharp
										: ClearSharp
								"
								:class="
									assessPlanWorkforce(
										poprData.FreeTechnician,
										poprData.OpenJobsTechnician,
										workforceData.technician.required
									)
										? 'text-positive'
										: 'text-negative'
								" />
						</PIcon>
					</td>
					<td>
						<PIcon>
							<component
								:is="
									assessPlanWorkforce(
										poprData.FreeEngineer,
										poprData.OpenJobsEngineer,
										workforceData.engineer.required
									)
										? CheckSharp
										: ClearSharp
								"
								:class="
									assessPlanWorkforce(
										poprData.FreeEngineer,
										poprData.OpenJobsEngineer,
										workforceData.engineer.required
									)
										? 'text-positive'
										: 'text-negative'
								" />
						</PIcon>
					</td>
					<td>
						<PIcon>
							<component
								:is="
									assessPlanWorkforce(
										poprData.FreeScientist,
										poprData.OpenJobsScientist,
										workforceData.scientist.required
									)
										? CheckSharp
										: ClearSharp
								"
								:class="
									assessPlanWorkforce(
										poprData.FreeScientist,
										poprData.OpenJobsScientist,
										workforceData.scientist.required
									)
										? 'text-positive'
										: 'text-negative'
								" />
						</PIcon>
					</td>
				</tr>
			</template>
		</tbody>
	</PTable>
</template>
