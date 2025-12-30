<script setup lang="ts">
	import { computed, ComputedRef, PropType, WritableComputedRef } from "vue";
	import { trackEvent } from "@/lib/analytics/useAnalytics";
	import { HabSolverGoal } from "@/features/planning/calculations/habOptimization";

	// Types & Interfaces
	import {
		IInfrastructureRecord,
		INFRASTRUCTURE_TYPE,
	} from "@/features/planning/usePlanCalculation.types";

	// UI
	import {
		PButton,
		PForm,
		PFormItem,
		PCheckbox,
		PInputNumber,
		PTooltip,
	} from "@/ui";

	const props = defineProps({
		disabled: {
			type: Boolean,
			required: true,
		},
		autoOptimizeHabs: {
			type: Boolean,
			required: true,
		},
		infrastructureData: {
			type: Object as PropType<IInfrastructureRecord>,
			required: true,
		},
		planetNaturalId: {
			type: String,
			required: true,
		},
	});

	const emit = defineEmits<{
		(
			e: "update:infrastructure",
			infrastructure: INFRASTRUCTURE_TYPE,
			value: number
		): void;
		(
			e: "update:auto-optimize-habs",
			value: boolean,
			goal: HabSolverGoal
		): void;
		(e: "optimize-habs", goal: HabSolverGoal): void;
	}>();

	// Local State
	const localInfrastructureData: ComputedRef<IInfrastructureRecord> =
		computed(() => props.infrastructureData);

	const infrastructureOrder: INFRASTRUCTURE_TYPE[] = [
		"HB1",
		"HBB",
		"HB2",
		"HBC",
		"HB3",
		"HBM",
		"HB4",
		"HBL",
		"HB5",
		"STO",
	];

	const localAutoOptimizeHabs: WritableComputedRef<boolean> = computed({
		get: () => props.autoOptimizeHabs,
		set: (value: boolean) => {
			emit("update:auto-optimize-habs", value, "auto");
		},
	});
</script>

<template>
	<div class="mb-3">
		<PForm>
			<PFormItem :label="$t('plan.infrastructure.auto_optimize')">
				<PTooltip>
					<template #trigger>
						<PCheckbox
							v-model:checked="localAutoOptimizeHabs"
							:disabled="disabled" />
					</template>
					{{ $t("plan.infrastructure.auto_optimize_tip") }}
				</PTooltip>
			</PFormItem>
		</PForm>
	</div>
	<div class="grid grid-cols-[repeat(4,auto)] gap-3 child:my-auto">
		<template v-for="inf in infrastructureOrder" :key="inf">
			<div>{{ inf }}</div>
			<PInputNumber
				v-model:value="localInfrastructureData[inf]"
				:disabled="disabled || (localAutoOptimizeHabs && inf !== 'STO')"
				show-buttons
				:min="0"
				class="min-w-[85px] max-w-[100px]"
				@update:value="
					(value) => {
						if (value !== null && value !== undefined) {
							emit('update:infrastructure', inf, value);
							trackEvent('plan_update_infrastructure', {
								planetNaturalId: props.planetNaturalId,
								infrastructureType: inf,
								amount: value,
							});
						}
					}
				" />
		</template>
		<div class="col-span-2 justify-self-center">
			<PButton
				:disabled="disabled || localAutoOptimizeHabs"
				@click="emit('optimize-habs', 'cost')">
				{{ $t("plan.infrastructure.optimize_cost") }}
			</PButton>
		</div>
		<div class="col-span-2 justify-self-center">
			<PButton
				:disabled="disabled || localAutoOptimizeHabs"
				@click="emit('optimize-habs', 'area')">
				{{ $t("plan.infrastructure.optimize_area") }}
			</PButton>
		</div>
	</div>
</template>
