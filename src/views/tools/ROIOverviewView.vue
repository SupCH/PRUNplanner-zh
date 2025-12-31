<script setup lang="ts">
	import { ref, Ref } from "vue";
	import { useHead } from "@unhead/vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Components
	import WrapperGameDataLoader from "@/features/wrapper/components/WrapperGameDataLoader.vue";
	import WrapperPlanningDataLoader from "@/features/wrapper/components/WrapperPlanningDataLoader.vue";
	import HelpDrawer from "@/features/help/components/HelpDrawer.vue";
	import CXPreferenceSelector from "@/features/exchanges/components/CXPreferenceSelector.vue";
	import ROIOverviewTable from "@/features/roi_overview/components/ROIOverviewTable.vue";

	useHead({
		title: `${t("tools.recipe_roi.title")} | PRUNplanner`,
	});

	// UI
	import { PForm, PFormItem } from "@/ui";

	/*
	 * ROI Overview assumes a fully standard planet, so we're going for Montem (OT-580b)
	 */
	const planetNaturalId: string = "OT-580b";
	const refSelectedCXUuid: Ref<string | undefined> = ref(undefined);
</script>

<template>
	<WrapperGameDataLoader
		load-exchanges
		load-materials
		load-buildings
		load-recipes>
		<WrapperPlanningDataLoader
			load-c-x
			:planet-natural-id="planetNaturalId"
			@update:cx-uuid="(d) => (refSelectedCXUuid = d)">
			<template #default="{ planDefinition }">
				<div class="min-h-screen flex flex-col">
					<div
						class="px-6 py-3 border-b border-white/10 flex flex-row justify-between">
						<h1 class="text-2xl font-bold my-auto">{{ $t("tools.recipe_roi.title") }}</h1>
						<div class="flex flex-row gap-x-3">
							<PForm>
								<PFormItem :label="$t('tools.recipe_roi.cx_preference')">
									<CXPreferenceSelector
										:cx-uuid="refSelectedCXUuid"
										@update:cxuuid="
											(value) =>
												(refSelectedCXUuid = value)
										" />
								</PFormItem>
							</PForm>
							<HelpDrawer file-name="tools_roi_overview" />
						</div>
					</div>

					<div class="px-6 py-3">
						<ROIOverviewTable
							v-if="planDefinition"
							:plan-definition="planDefinition"
							:cx-uuid="refSelectedCXUuid" />
					</div>
				</div>
			</template>
		</WrapperPlanningDataLoader>
	</WrapperGameDataLoader>
</template>
