<script setup lang="ts">
	import { computed, PropType } from "vue";

	// Types & Interfaces
	import { PSelectOption } from "@/ui/ui.types";

	// UI
	import { PSelectMultiple } from "@/ui";

	const props = defineProps({
		filterPlanNames: {
			type: Array as PropType<string[]>,
			required: true,
		},
		filterEmpires: {
			type: Array as PropType<string[]>,
			required: true,
		},
		optionsPlanNames: {
			type: Array as PropType<PSelectOption[]>,
			required: true,
		},
		optionsEmpires: {
			type: Array as PropType<PSelectOption[]>,
			required: true,
		},
	});

	const emit = defineEmits<{
		(e: "update:filterPlanNames", value: string[]): void;
		(e: "update:filterEmpires", value: string[]): void;
	}>();

	const localFilterPlanNames = computed({
		get: () => props.filterPlanNames,
		set: (v: string[]) => emit("update:filterPlanNames", v),
	});

	const localFilterEmpires = computed({
		get: () => props.filterEmpires,
		set: (v: string[]) => emit("update:filterEmpires", v),
	});
</script>

<template>
	<div
		class="border border-b-0 rounded-[3px] border-white/15 p-3 flex flex-row gap-3 flex-wrap">
		<div class="my-auto font-bold pr-3">{{ $t("manage.assignments.filters.heading") }}</div>
		<div class="flex flex-row gap-3 child:my-auto">
			<div>{{ $t("manage.assignments.filters.plan_label") }}</div>
			<div>
				<PSelectMultiple
					v-model:value="localFilterPlanNames"
					:options="optionsPlanNames"
					searchable
					clearable
					class="!w-[400px]" />
			</div>
		</div>
		<div class="flex flex-row gap-3 child:my-auto">
			<div>{{ $t("manage.assignments.filters.empire_label") }}</div>
			<div>
				<PSelectMultiple
					v-model:value="localFilterEmpires"
					:options="optionsEmpires"
					searchable
					clearable
					class="!w-[400px]" />
			</div>
		</div>
	</div>
</template>
