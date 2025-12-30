<script setup lang="ts">
	import { PropType, ref, Ref, watch, computed } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Composables
	import { useQuery } from "@/lib/query_cache/useQuery";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Util
	import { inertClone } from "@/util/data";

	// Types & Interfaces
	import {
		IPlanEmpireElement,
		PLAN_FACTION,
	} from "@/stores/planningStore.types";
	import { IEmpirePatchPayload } from "@/features/empire/empire.types";
	import { PSelectOption } from "@/ui/ui.types";

	// UI
	import {
		PButton,
		PCheckbox,
		PForm,
		PFormItem,
		PInputNumber,
		PInput,
		PSelect,
	} from "@/ui";
	import { SaveSharp, ChangeCircleOutlined } from "@vicons/material";

	const props = defineProps({
		data: {
			type: Object as PropType<IPlanEmpireElement>,
			required: true,
		},
	});

	const emit = defineEmits<{
		(e: "reload:empires"): void;
	}>();

	const isLoading: Ref<boolean> = ref(false);

	// Local Data & Watcher
	const localData: Ref<IPlanEmpireElement> = ref(inertClone(props.data));

	watch(
		() => props.data,
		(newData: IPlanEmpireElement) =>
			(localData.value = inertClone(newData)),
		{ deep: true }
	);

	const factionOptions = computed<PSelectOption[]>(() => [
		{ label: t("empire.config.factions.NONE"), value: "NONE" },
		{ label: t("empire.config.factions.ANTARES"), value: "ANTARES" },
		{ label: t("empire.config.factions.BENTEN"), value: "BENTEN" },
		{ label: t("empire.config.factions.HORTUS"), value: "HORTUS" },
		{ label: t("empire.config.factions.MORIA"), value: "MORIA" },
		{ label: t("empire.config.factions.OUTSIDEREGION"), value: "OUTSIDEREGION" },
	]);

	/**
	 * Reloads data from props again
	 * @author jplacht
	 *
	 * @returns {void}
	 */
	function reload(): void {
		trackEvent("empire_reload");
		localData.value = inertClone(props.data);
	}

	/**
	 * Persists data against backend api, triggers reload emit
	 * @author jplacht
	 *
	 * @async
	 * @returns {Promise<void>}
	 */
	async function save(): Promise<void> {
		isLoading.value = true;
		trackEvent("empire_patch");

		const patchData: IEmpirePatchPayload = {
			faction: localData.value.faction as PLAN_FACTION,
			permits_used: localData.value.permits_used,
			permits_total: localData.value.permits_total,
			name: localData.value.name,
			use_fio_storage: localData.value.use_fio_storage,
		};

		try {
			await useQuery("PatchEmpire", {
				empireUuid: localData.value.uuid,
				data: patchData,
			}).execute();
			emit("reload:empires");
		} catch (err) {
			console.error("Error patching empire", err);
		} finally {
			isLoading.value = false;
		}
	}
</script>

<template>
	<div class="p-3 child:my-auto border border-white/10 rounded">
		<div class="pb-3 flex justify-between child:my-auto">
			<h2 class="flex-grow text-white/80 font-bold text-lg">
				{{ $t("empire.config.heading") }}
			</h2>

			<div class="flex gap-x-3">
				<PButton size="md" :loading="isLoading" @click="save">
					<template #icon><SaveSharp /></template>
					{{ $t("plan.actions.save") }}
				</PButton>
				<PButton size="md" @click="reload">
					<template #icon><ChangeCircleOutlined /></template>
					{{ $t("plan.actions.reload") }}
				</PButton>
			</div>
		</div>
		<PForm>
			<PFormItem :label="$t('empire.config.name')">
				<PInput v-model:value="localData.name" class="w-full" />
			</PFormItem>
			<PFormItem :label="$t('empire.config.faction')">
				<PSelect
					v-model:value="localData.faction"
					class="w-full"
					:options="factionOptions" />
			</PFormItem>
			<PFormItem :label="$t('empire.config.permits_total')">
				<PInputNumber
					v-model:value="localData.permits_total"
					show-buttons
					:min="2" />
			</PFormItem>
			<PFormItem :label="$t('empire.config.permits_used')">
				<PInputNumber
					v-model:value="localData.permits_used"
					show-buttons
					:min="1" />
			</PFormItem>
			<PFormItem :label="$t('empire.config.use_fio_storage')">
				<PCheckbox v-model:checked="localData.use_fio_storage" />
			</PFormItem>
		</PForm>
	</div>
</template>
