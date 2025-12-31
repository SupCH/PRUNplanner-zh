<script setup lang="ts">
	import { computed, onMounted, ref, Ref, watch } from "vue";

	import { useHead } from "@unhead/vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	useHead({
		title: `${t("api.title")} | PRUNplanner`,
	});

	// Components
	import WrapperPlanningDataLoader from "@/features/wrapper/components/WrapperPlanningDataLoader.vue";
	import APIKeyManagement from "@/features/endpoints/components/APIKeyManagement.vue";
	import APIEndpointList from "@/features/endpoints/components/APIEndpointList.vue";

	// Composables
	import { useEndpoints } from "@/features/endpoints/useEndpoints";

	// Types & Interfaces
	import { IUserAPIKey } from "@/features/api/userData.types";

	// UI
	import { PForm, PFormItem, PSelect, PSpin } from "@/ui";
	import { IPlanEmpireElement } from "@/stores/planningStore.types";

	const { isLoading, loadAPIKeys } = useEndpoints();

	const apiKeys: Ref<IUserAPIKey[]> = ref([]);
	const empireList: Ref<IPlanEmpireElement[]> = ref([]);
	const selectedAPIKey: Ref<string | null> = ref(null);
	const selectedEmpireUuid: Ref<string | null> = ref(null);

	async function loadAndStoreAPIKeys() {
		loadAPIKeys().then((keys) => (apiKeys.value = keys));
	}

	const empireSelectOptions = computed(() =>
		empireList.value.map((e) => ({ label: e.name, value: e.uuid }))
	);

	const apiKeySelectOptions = computed(() =>
		apiKeys.value.map((a) => ({ label: a.name, value: a.key }))
	);

	watch(
		() => apiKeys.value,
		() =>
			apiKeys.value.length > 0
				? (selectedAPIKey.value = apiKeys.value[0].key)
				: (selectedAPIKey.value = null),
		{ immediate: true }
	);

	watch(
		() => empireList.value,
		() =>
			empireList.value.length > 0
				? (selectedEmpireUuid.value = empireList.value[0].uuid)
				: (selectedEmpireUuid.value = null),
		{ immediate: true }
	);

	onMounted(() => loadAndStoreAPIKeys());
</script>

<template>
	<WrapperPlanningDataLoader
		empire-list
		@data:empire:list="(v) => (empireList = v)">
		<div class="min-h-screen flex flex-col">
			<div
				class="px-6 py-3 border-b border-white/10 flex flex-row justify-between">
				<h1 class="text-2xl font-bold">{{ $t("api.title") }}</h1>
				<PSpin v-if="isLoading" />
			</div>
			<div
				class="flex-grow grid grid-cols-1 xl:grid-cols-2 gap-3 divide-x divide-white/10 child:px-6 child:py-3 xl:child:last:pl-3">
				<div>
					<div class="pb-3 text-white/60">
						{{ $t("api.description") }}
					</div>
					<APIKeyManagement
						:api-keys="apiKeys"
						@reload:keys="loadAndStoreAPIKeys" />
				</div>
				<div>
					<div class="flex flex-row flex-wrap justify-between pb-3">
						<div>
							<PForm>
								<PFormItem :label="$t('api.api_key_label')">
									<PSelect
										v-model:value="selectedAPIKey"
										:options="apiKeySelectOptions"
										:placeholder="$t('api.api_key_label')"
										class="w-[200px]" />
								</PFormItem>
							</PForm>
						</div>
						<div>
							<PForm>
								<PFormItem :label="$t('api.empire_label')">
									<PSelect
										v-model:value="selectedEmpireUuid"
										:options="empireSelectOptions"
										clearable
										:placeholder="$t('api.empire_label')"
										class="w-[200px]" />
								</PFormItem>
							</PForm>
						</div>
					</div>
					<APIEndpointList
						:selected-key="selectedAPIKey"
						:selected-empire-uuid="selectedEmpireUuid" />
				</div>
			</div>
		</div>
	</WrapperPlanningDataLoader>
</template>
