<script setup lang="ts">
	import { computed, PropType, WritableComputedRef } from "vue";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Types & Interfaces
	import { IAreaResult } from "@/features/planning/usePlanCalculation.types";

	// UI
	import { PForm, PFormItem, PInputNumber } from "@/ui";

	const props = defineProps({
		disabled: {
			type: Boolean,
			required: true,
		},
		areaData: {
			type: Object as PropType<IAreaResult>,
			required: true,
		},
		planetNaturalId: {
			type: String,
			required: true,
		},
	});

	const emit = defineEmits<{
		(e: "update:permits", value: number): void;
	}>();

	// Local State
	const localPermits: WritableComputedRef<number> = computed({
		get: () => props.areaData.permits,
		set: (value: number) => {
			emit("update:permits", value);
			trackEvent("plan_update_permits", {
				permits: value,
				planetNaturalId: props.planetNaturalId,
			});
		},
	});
</script>

<template>
	<PForm>
		<PFormItem :label="$t('plan.area.permits')">
			<PInputNumber
				v-model:value="localPermits"
				:disabled="disabled"
				show-buttons
				:min="1"
				:max="3"
				class="w-full" />
		</PFormItem>
		<PFormItem :label="$t('plan.area.area')">
			<div class="flex flex-row w-full">
				<div class="flex-grow">
					{{ areaData.areaUsed }} / {{ areaData.areaTotal }}
				</div>
				<div>
					<span
						class="font-bold"
						:class="
							areaData.areaLeft >= 0
								? 'text-positive'
								: 'text-negative'
						"
						>{{ areaData.areaLeft }}</span
					>
					{{ $t("plan.area.free") }}
				</div>
			</div>
		</PFormItem>
	</PForm>
</template>
