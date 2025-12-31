<script setup lang="ts">
	import {
		computed,
		ComputedRef,
		defineAsyncComponent,
		ref,
		Ref,
		watch,
	} from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Stores
	import { usePlanningStore } from "@/stores/planningStore";

	// Composables
	import { useQuery } from "@/lib/query_cache/useQuery";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Util
	import { inertClone } from "@/util/data";

	// Components
	import WrapperPlanningDataLoader from "@/features/wrapper/components/WrapperPlanningDataLoader.vue";
	import WrapperGameDataLoader from "@/features/wrapper/components/WrapperGameDataLoader.vue";
	import HelpDrawer from "@/features/help/components/HelpDrawer.vue";
	const AsyncWrapperGenericError = defineAsyncComponent(
		() => import("@/features/wrapper/components/WrapperGenericError.vue")
	);

	import CXExchangePreference from "@/features/exchanges/components/CXExchangePreference.vue";
	import CXTickerPreference from "@/features/exchanges/components/CXTickerPreference.vue";
	import CXPlanetPreferenceTable from "@/features/exchanges/components/CXPlanetPreferenceTable.vue";

	// Types & Interfaces
	import { ICX, ICXData } from "@/stores/planningStore.types";
	import { ICXPlanetMap } from "@/features/exchanges/manageCX.types";

	// UI
	import { PButton, PInput, PIcon } from "@/ui";
	import { NDropdown } from "naive-ui";
	import {
		ArrowDropDownSharp,
		SaveSharp,
		ChangeCircleOutlined,
	} from "@vicons/material";

	// Unhead
	import { useHead } from "@unhead/vue";

	useHead({
		title: `${t("manage.exchanges.title")} | PRUNplanner`,
	});

	const planningStore = usePlanningStore();

	const props = defineProps({
		cxUuid: {
			type: String,
			required: false,
			default: undefined,
		},
	});

	const localCXUuid: Ref<string | undefined> = ref(props.cxUuid);
	const localCXs: Ref<ICX[]> = ref([]);
	const selectedCX: Ref<ICX | null> = ref(null);
	const selectedName: Ref<string | null> = ref(null);
	const rawSelectedCX: Ref<ICX | null> = ref(null);
	const planetMap: Ref<ICXPlanetMap> = ref({});

	const selectorDropdownOptions = computed(() =>
		localCXs.value.map((c) => ({
			label: c.name,
			key: c.uuid,
		}))
	);

	const cxName = computed(() => {
		if (!localCXUuid.value) return t("manage.exchanges.title");
		return planningStore.getCX(localCXUuid.value).name;
	});

	const localPlanetList: Ref<string[]> = ref([]);

	watch(
		[() => localCXUuid.value, () => localPlanetList.value],
		([cxUuid, planetList]) => {
			if (cxUuid) {
				initialize(cxUuid);
				if (planetList.length > 0) {
					generatePlanetMap(planetList);
				}
			}
		},
		{ immediate: true }
	);

	function initialize(cxUuid: string): void {
		selectedCX.value = planningStore.getCX(cxUuid);
		selectedName.value = selectedCX.value.name;

		// save raw, in order to re-use on "reload" button
		rawSelectedCX.value = planningStore.getCX(cxUuid);
	}

	function generatePlanetMap(planetList: string[]): void {
		planetMap.value = planetList.reduce((acc, planet) => {
			acc[planet] = {
				planet: planet,
				exchanges:
					selectedCX.value?.cx_data.cx_planets.find(
						(e) => e.planet === planet
					)?.preferences ?? [],
				ticker:
					selectedCX.value?.cx_data.ticker_planets.find(
						(e) => e.planet === planet
					)?.preferences ?? [],
			};
			return acc;
		}, {} as ICXPlanetMap);
	}

	const patchData: ComputedRef<undefined | ICXData> = computed(() => {
		const activeCX = selectedCX.value;
		if (activeCX && selectedName.value && selectedName.value != "") {
			const patch = {
				name: selectedName.value,
				cx_empire: activeCX.cx_data.cx_empire,
				ticker_empire: activeCX.cx_data.ticker_empire,
				cx_planets: Object.values(planetMap.value)
					.map((p) => {
						return {
							planet: p.planet,
							preferences: p.exchanges,
						};
					})
					.filter((f) => f.preferences.length > 0),
				ticker_planets: Object.values(planetMap.value)
					.map((p) => {
						return {
							planet: p.planet,
							preferences: p.ticker,
						};
					})
					.filter((f) => f.preferences.length > 0),
			};
			return patch;
		}
		return undefined;
	});

	const isPatching: Ref<boolean> = ref(false);
	async function patchCX(data: ICXData): Promise<void> {
		if (selectedCX.value) {
			isPatching.value = true;

			trackEvent("exchange_patch", {
				cxUuid: selectedCX.value.uuid,
				location: "exchanges_view",
			});

			await useQuery("PatchCX", {
				cxUuid: selectedCX.value.uuid,
				data: data,
			}).execute();

			initialize(selectedCX.value!.uuid);
			isPatching.value = false;
		}
	}

	function reloadCXData(): void {
		trackEvent("exchange_reload", { location: "exchanges_view" });
		selectedCX.value = inertClone(rawSelectedCX.value);
		selectedName.value = selectedCX.value!.name;
		generatePlanetMap(localPlanetList.value);
	}
</script>

<template>
	<WrapperPlanningDataLoader
		plan-list
		load-c-x
		:cx-uuid="props.cxUuid"
		@update:cx-uuid="(cxUuid: string | undefined) => localCXUuid = cxUuid"
		@data:cx="(data: ICX[]) => localCXs = data"
		@data:plan:list:planets="(data: string[]) => localPlanetList = data">
		<WrapperGameDataLoader load-exchanges load-materials>
			<template v-if="!localCXUuid">
				<AsyncWrapperGenericError
					:message-title="$t('manage.exchanges.no_preferences_title')"
					:message-text="$t('manage.exchanges.no_preferences_text')" />
			</template>
			<div v-else class="min-h-screen flex flex-col">
				<div
					class="px-6 py-3 border-b border-white/10 flex flex-row justify-between gap-x-3">
					<h1 class="text-2xl font-bold my-auto hover:cursor-pointer">
						<n-dropdown
							v-if="selectorDropdownOptions.length > 0"
							trigger="hover"
							:options="selectorDropdownOptions"
							@select="(value: string) => localCXUuid = value">
							<div>
								<PIcon class="-mr-1">
									<ArrowDropDownSharp />
								</PIcon>
								{{ cxName }}
							</div>
						</n-dropdown>
						<template v-else>{{ $t("manage.exchanges.title") }}</template>
					</h1>
					<div class="flex flex-row gap-x-3">
						<PButton
							v-if="patchData"
							:loading="isPatching"
							@click="patchCX(patchData)">
							<template #icon><SaveSharp /></template>
							{{ $t("plan.actions.save") }}
						</PButton>
						<PButton @click="reloadCXData">
							<template #icon><ChangeCircleOutlined /></template>
							{{ $t("plan.actions.reload") }}
						</PButton>
						<HelpDrawer file-name="exchanges" />
					</div>
				</div>
				<div
					:kex="`EXCHANGE#${localCXUuid}`"
					class="flex-grow grid grid-cols-1 lg:grid-cols-[25%_auto] divide-x divide-white/10">
					<div class="px-6 pb-3 pt-4 border-b border-white/10">
						<h3 class="text-lg font-bold pb-3">{{ $t("manage.exchanges.preference_name") }}</h3>
						<PInput
							v-model:value="selectedName"
							:status="
								!selectedName || selectedName === ''
									? 'warning'
									: 'success'
							" />
						<h2 class="text-xl font-bold py-3 pt-6 my-auto">
							{{ $t("manage.exchanges.empire_preferences") }}
						</h2>
						<h3 class="text-lg font-bold pb-3">{{ $t("manage.exchanges.exchange_label") }}</h3>
						<CXExchangePreference
							v-if="selectedCX"
							v-model:cx-options="selectedCX.cx_data.cx_empire" />
						<h3 class="text-lg font-bold py-3">{{ $t("manage.exchanges.ticker_label") }}</h3>
						<CXTickerPreference
							v-if="selectedCX"
							v-model:cx-options="
								selectedCX.cx_data.ticker_empire
							" />
					</div>
					<div class="p-6">
						<CXPlanetPreferenceTable
							v-if="selectedCX"
							:key="selectedCX.uuid"
							:planet-map="planetMap" />
					</div>
				</div>
			</div>
		</WrapperGameDataLoader>
	</WrapperPlanningDataLoader>
</template>
