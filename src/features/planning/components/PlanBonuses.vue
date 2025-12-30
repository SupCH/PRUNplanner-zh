<script setup lang="ts">
	import { computed, PropType, WritableComputedRef } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();
	import { trackEvent } from "@/lib/analytics/useAnalytics";
	// Types & Interfaces
	import { PLAN_COGCPROGRAM_TYPE } from "@/stores/planningStore.types";

	// UI
	import { PForm, PFormItem, PCheckbox, PSelect, PTooltip } from "@/ui";
	import { PSelectOption } from "@/ui/ui.types";

	const props = defineProps({
		disabled: {
			type: Boolean,
			required: true,
		},
		corphq: {
			type: Boolean,
			required: true,
		},
		cogc: {
			type: String as PropType<PLAN_COGCPROGRAM_TYPE>,
			required: true,
		},
		planetNaturalId: {
			type: String,
			required: true,
		},
	});

	const emit = defineEmits<{
		(e: "update:corphq", value: boolean): void;
		(e: "update:cogc", value: PLAN_COGCPROGRAM_TYPE): void;
	}>();

	// Local State
	const localCorpHQ: WritableComputedRef<boolean> = computed({
		get: () => props.corphq,
		set: (value: boolean) => {
			emit("update:corphq", value);
			trackEvent("plan_update_corphq", {
				planetNaturalId: props.planetNaturalId,
				corphq: value,
			});
		},
	});

	const localCOGC: WritableComputedRef<PLAN_COGCPROGRAM_TYPE> = computed({
		get: () => props.cogc,
		set: (value: PLAN_COGCPROGRAM_TYPE) => {
			emit("update:cogc", value);
			trackEvent("plan_update_cogc", {
				planetNaturalId: props.planetNaturalId,
				cogc: value,
			});
		},
	});

	const cogcOptions = computed<PSelectOption[]>(() => [
		{ value: "---", label: t("empire.plan_list.cogc_mapping.---") },
		{ value: "AGRICULTURE", label: t("plan.experts.mapping.AGRICULTURE") },
		{ value: "CHEMISTRY", label: t("plan.experts.mapping.CHEMISTRY") },
		{ value: "CONSTRUCTION", label: t("plan.experts.mapping.CONSTRUCTION") },
		{ value: "ELECTRONICS", label: t("plan.experts.mapping.ELECTRONICS") },
		{ value: "FOOD_INDUSTRIES", label: t("plan.experts.mapping.FOOD_INDUSTRIES") },
		{ value: "FUEL_REFINING", label: t("plan.experts.mapping.FUEL_REFINING") },
		{ value: "MANUFACTURING", label: t("plan.experts.mapping.MANUFACTURING") },
		{ value: "METALLURGY", label: t("plan.experts.mapping.METALLURGY") },
		{ value: "RESOURCE_EXTRACTION", label: t("plan.experts.mapping.RESOURCE_EXTRACTION") },
		{ value: "PIONEERS", label: t("empire.plan_list.cogc_mapping.PIONEERS") },
		{ value: "SETTLERS", label: t("empire.plan_list.cogc_mapping.SETTLERS") },
		{ value: "TECHNICIANS", label: t("empire.plan_list.cogc_mapping.TECHNICIANS") },
		{ value: "ENGINEERS", label: t("empire.plan_list.cogc_mapping.ENGINEERS") },
		{ value: "SCIENTISTS", label: t("empire.plan_list.cogc_mapping.SCIENTISTS") },
	]);
</script>

<template>
	<PForm>
		<PFormItem :label="$t('plan.bonuses.corp_hq')">
			<PTooltip>
				<template #trigger>
					<PCheckbox
						v-model:checked="localCorpHQ"
						:disabled="disabled" />
				</template>
				{{ $t("plan.bonuses.corp_hq_tip") }}
			</PTooltip>
		</PFormItem>

		<PFormItem :label="$t('plan.bonuses.cogc')">
			<PSelect
				v-model:value="localCOGC"
				class="w-full"
				:disabled="disabled"
				:options="cogcOptions" />
		</PFormItem>
	</PForm>
</template>
