<script setup lang="ts">
	import { onMounted, ref, Ref } from "vue";

	// Stores
	import { usePlanningStore } from "@/stores/planningStore";

	// Composables
	import { usePreferences } from "@/features/preferences/usePreferences";

	// Constants
	import { XITSTATIONWAREHOUSES } from "@/features/xit/xitConstants";

	// Types & Interfaces
	import { PSelectOption } from "@/ui/ui.types";

	// Components
	import CXPreferenceSelector from "@/features/exchanges/components/CXPreferenceSelector.vue";

	// UI
	import {
		PForm,
		PFormItem,
		PFormSeperator,
		PSelect,
		PInputNumber,
		PCheckbox,
		PTable,
	} from "@/ui";

	const planningStore = usePlanningStore();

	const {
		burnDaysRed,
		burnDaysYellow,
		burnResupplyDays,
		burnOrigin,
		planSettingsOverview,
		cleanPlanPreferences,
	} = usePreferences();
	let { defaultEmpireUuid, defaultCXUuid, defaultBuyItemsFromCX } =
		usePreferences();

	const empireOptions: Ref<PSelectOption[]> = ref(
		Object.values(planningStore.empires).map((e) => {
			return {
				label: e.name,
				value: e.uuid,
			};
		})
	);

	onMounted(() => {
		// ensure defaultEmpire is still in the empire list
		const validDefaultEmpire: boolean = empireOptions.value.find(
			(e) => e.value === defaultEmpireUuid.value
		)
			? true
			: false;
		if (!validDefaultEmpire) {
			// if there empires, use first
			if (empireOptions.value.length > 0)
				defaultEmpireUuid.value = empireOptions.value[0]
					.value as string;
			// or, reset to undefined
			else defaultEmpireUuid.value = undefined;
		}

		// all plans loaded, clear up non-existing plans preferences
		cleanPlanPreferences();
	});
</script>

<template>
	<h2 class="text-white/80 font-bold text-lg my-auto">{{ $t("profile.preferences.title") }}</h2>
	<div class="py-3 text-white/60">
		{{ $t("profile.preferences.description") }}
	</div>

	<PForm>
		<PFormSeperator>
			<h3 class="font-bold pb-3">{{ $t("profile.preferences.tools_heading") }}</h3>
		</PFormSeperator>
		<PFormItem :label="$t('profile.preferences.default_empire')">
			<PSelect
				v-model:value="defaultEmpireUuid"
				:options="empireOptions"
				class="w-full"
				@update:value="
					(value) => {
						if (value && typeof value === 'string') {
							defaultEmpireUuid = value;
						}
					}
				" />
		</PFormItem>
		<PFormItem :label="$t('profile.preferences.default_cx')">
			<CXPreferenceSelector
				:cx-uuid="defaultCXUuid"
				:add-undefined-c-x="false"
				class="w-full" />
		</PFormItem>

		<PFormSeperator>
			<h4 class="font-bold py-1">{{ $t("nav.items.fio_burn") }}</h4>
		</PFormSeperator>

		<PFormItem :label="$t('profile.preferences.burn_red')">
			<PInputNumber
				v-model:value="burnDaysRed"
				show-button
				:min="1"
				class="w-full" />
		</PFormItem>
		<PFormItem :label="$t('profile.preferences.burn_yellow')">
			<PInputNumber
				v-model:value="burnDaysYellow"
				show-button
				:min="1"
				class="w-full" />
		</PFormItem>
		<PFormItem :label="$t('profile.preferences.resupply_days')">
			<PInputNumber
				v-model:value="burnResupplyDays"
				show-button
				:min="1"
				class="w-full" />
		</PFormItem>
		<PFormItem :label="$t('profile.preferences.xit_origin')">
			<PSelect
				v-model:value="burnOrigin"
				:options="XITSTATIONWAREHOUSES"
				class="w-full" />
		</PFormItem>
		<PFormItem :label="$t('profile.preferences.xit_buy_cx')">
			<PCheckbox v-model:checked="defaultBuyItemsFromCX" />
		</PFormItem>
	</PForm>

	<h3 class="font-bold py-3">{{ $t("profile.preferences.plan_settings_heading") }}</h3>
	<div class="pb-3 text-white/60">
		{{ $t("profile.preferences.plan_settings_description") }}
	</div>
	<PTable striped>
		<thead>
			<tr>
				<th>{{ $t("profile.preferences.table.plan") }}</th>
				<th>{{ $t("profile.preferences.table.preferences") }}</th>
			</tr>
		</thead>
		<tbody>
			<tr v-for="plan in planSettingsOverview" :key="plan.planUuid">
				<td>
					<router-link
						:to="`/plan/${plan.planetId}/${plan.planUuid}`"
						class="text-link-primary font-bold hover:underline">
						{{ plan.planName }}
					</router-link>
				</td>
				<td class="w-[50%] max-w-[75%]">
					{{ plan.preferences.join(", ") }}
				</td>
			</tr>
		</tbody>
	</PTable>
</template>
