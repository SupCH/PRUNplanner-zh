<script setup lang="ts">
	import { computed, PropType } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Types & Interfaces
	import {
		IAreaResult,
		IExpertRecord,
		IOverviewData,
	} from "@/features/planning/usePlanCalculation.types";
	import { PLAN_COGCPROGRAM_TYPE } from "@/stores/planningStore.types";
	import { cogcTextMapping } from "@/features/planning_data/usePlan";

	// Util
	import { formatNumber } from "@/util/numbers";

	// UI
	import {} from "@/ui";

	const props = defineProps({
		areaData: {
			type: Object as PropType<IAreaResult>,
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
		expertData: {
			type: Object as PropType<IExpertRecord>,
			required: true,
		},
		overviewData: {
			type: Object as PropType<IOverviewData>,
			required: true,
		},
	});

	const expertsString = computed(() => {
		let experts = "";

		for (const expertKey in props.expertData) {
			const expert = props.expertData[expertKey as keyof IExpertRecord];
			if (expert.amount > 0) {
				if (experts.length > 0) experts += ", ";
				experts += `${expert.amount}x${t("plan.experts.mapping." + expert.name.toUpperCase()).substring(0, 4)}`;
			}
		}
		if (experts.length === 0) experts = t("plan.status_bar.none");
		return experts;
	});
</script>

<template>
	<div class="flex flex-row font-bold child:mr-3">
		<div :class="corphq ? 'visible' : 'collapse md:invisible'">
			<span class="text-positive">HQ</span>
		</div>
		<div>
			<span>COGC: </span>
			<span :class="props.cogc === '---' ? 'text-negative' : ''">
				{{ $t("empire.plan_list.cogc_mapping." + props.cogc) }}
			</span>
		</div>
		<div>
			<span>{{ $t("plan.status_bar.area") }}: </span>
			<span
				:class="
					areaData.areaUsed > areaData.areaTotal
						? 'text-negative'
						: ''
				">
				{{ areaData.areaUsed }}
			</span>
			<span>/{{ areaData.areaTotal }}</span>
		</div>
		<div>
			<span>{{ $t("plan.status_bar.profit") }}: </span>
			<span
				:class="
					overviewData.profit > 0 ? 'text-positive' : 'text-negative'
				">
				{{ formatNumber(overviewData.profit) }}
			</span>
		</div>
		<div>
			<span>{{ $t("plan.status_bar.experts") }}: </span>
			<span :class="expertsString === $t('plan.status_bar.none') ? 'text-negative' : ''">
				{{ expertsString }}
			</span>
		</div>
	</div>
</template>
