<script setup lang="ts">
	import { computed, ComputedRef, PropType } from "vue";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Utils
	import { capitalizeString } from "@/util/text";
	import { formatAmount, formatNumber } from "@/util/numbers";

	// Types & Interfaces
	import {
		IWorkforceRecord,
		WORKFORCE_TYPE,
	} from "@/features/planning/usePlanCalculation.types";

	// UI
	import { PButton, PTable } from "@/ui";
	import { CheckSharp, RadioButtonUncheckedSharp } from "@vicons/material";

	const props = defineProps({
		disabled: {
			type: Boolean,
			required: true,
		},
		workforceData: {
			type: Object as PropType<IWorkforceRecord>,
			required: true,
		},
		planetNaturalId: {
			type: String,
			required: true,
		},
	});

	const emit = defineEmits<{
		(
			e: "update:lux",
			workforce: WORKFORCE_TYPE,
			luxType: "lux1" | "lux2",
			value: boolean
		): void;
	}>();

	// Local State
	const localWorkforceData: ComputedRef<IWorkforceRecord> = computed(
		() => props.workforceData
	);

	// Click Handlers
	function updateLux(
		workforce: WORKFORCE_TYPE,
		luxType: "lux1" | "lux2",
		value: boolean
	) {
		emit("update:lux", workforce, luxType, value);
		trackEvent("plan_update_workforce", {
			planetNaturalId: props.planetNaturalId,
			workforceType: workforce,
			luxType: luxType === "lux1" ? "Lux1" : "Lux2",
			value: value,
		});
	}
</script>

<template>
	<PTable striped>
		<thead>
			<tr>
				<th>{{ $t("plan.workforce.type") }}</th>
				<th>{{ $t("plan.workforce.required") }}</th>
				<th>{{ $t("plan.workforce.capacity") }}</th>
				<th>{{ $t("plan.workforce.open") }}</th>
				<th class="!text-center">L1</th>
				<th class="!text-center">L2</th>
				<th class="!text-end">{{ $t("plan.workforce.efficiency") }}</th>
			</tr>
		</thead>
		<tbody>
			<tr v-for="workforce in localWorkforceData" :key="workforce.name">
				<td class="font-bold">
					{{ $t("empire.plan_list.cogc_mapping." + workforce.name.toUpperCase()) }}
				</td>
				<td :class="workforce.required === 0 ? '!text-white/50' : ''">
					{{ formatAmount(workforce.required) }}
				</td>
				<td :class="workforce.capacity === 0 ? '!text-white/50' : ''">
					{{ formatAmount(workforce.capacity) }}
				</td>
				<td :class="workforce.left === 0 ? '!text-white/50' : ''">
					{{ formatAmount(workforce.left) }}
				</td>
				<td class="text-center">
					<PButton
						:disabled="disabled"
						size="sm"
						:type="workforce.lux1 ? 'success' : 'error'"
						@click="
							updateLux(workforce.name, 'lux1', !workforce.lux1)
						">
						<template #icon>
							<CheckSharp v-if="workforce.lux1" />
							<RadioButtonUncheckedSharp v-else />
						</template>
					</PButton>
				</td>
				<td class="text-center">
					<PButton
						:disabled="disabled"
						size="sm"
						:type="workforce.lux2 ? 'success' : 'error'"
						@click="
							updateLux(workforce.name, 'lux2', !workforce.lux2)
						">
						<template #icon>
							<CheckSharp v-if="workforce.lux2" />
							<RadioButtonUncheckedSharp v-else />
						</template>
					</PButton>
				</td>
				<td
					class="text-end"
					:class="workforce.efficiency === 0 ? '!text-white/50' : ''">
					{{ formatNumber(workforce.efficiency * 100) }} %
				</td>
			</tr>
		</tbody>
	</PTable>
</template>
