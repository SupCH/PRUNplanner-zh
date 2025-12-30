<script setup lang="ts">
	import { computed, ComputedRef, PropType, ref, Ref, watch } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Composables
	import { useQuery } from "@/lib/query_cache/useQuery";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Types & Interfaces
	import {
		ICX,
		IPlanEmpireElement,
		PLAN_FACTION,
	} from "@/stores/planningStore.types";
	import { ICXEmpireJunction } from "@/features/manage/manage.types";
	import { PSelectOption } from "@/ui/ui.types";

	// Util
	import { inertClone } from "@/util/data";
	import { capitalizeString } from "@/util/text";

	// UI
	import {
		PForm,
		PFormItem,
		PButton,
		PInput,
		PInputNumber,
		PSelect,
		PCheckbox,
		PTag,
	} from "@/ui";
	import { useDialog } from "naive-ui";
	const dialog = useDialog();
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";
	import { ClearSharp, PlusSharp, SaveSharp } from "@vicons/material";

	const props = defineProps({
		empires: {
			type: Array as PropType<IPlanEmpireElement[]>,
			required: true,
		},
		cx: {
			type: Array as PropType<ICX[]>,
			required: true,
		},
	});

	// Local Data & Watcher
	const localEmpires: ComputedRef<IPlanEmpireElement[]> = computed(() =>
		inertClone(props.empires)
	);
	const localCX: ComputedRef<ICX[]> = computed(() => inertClone(props.cx));

	watch([() => props.empires, () => props.cx], () => {
		generateCXOptions();
		generateEmpireCXMap();
	});

	const emit = defineEmits<{
		(e: "update:cxList", value: ICX[]): void;
		(e: "update:empireList", value: IPlanEmpireElement[]): void;
	}>();

	const refCXOptions: Ref<PSelectOption[]> = ref([]);
	const refEmpireCXMap: Ref<Record<string, string | undefined>> = ref({});
	const refIsUpdatingJunctions: Ref<boolean> = ref(false);
	const refShowCreateEmpire: Ref<boolean> = ref(false);

	const refCreateFaction: Ref<PLAN_FACTION> = ref("NONE");
	const refCreatePermitsUsed: Ref<number> = ref(1);
	const refCreatePermitsTotal: Ref<number> = ref(2);
	const refCreateName: Ref<string | undefined> = ref(undefined);
	const refCreateUseFioStorage: Ref<boolean> = ref(false);
	const refIsCreating: Ref<boolean> = ref(false);
	const refIsDeleting: Ref<string | undefined> = ref(undefined);
	const compCanCreate: ComputedRef<boolean> = computed(() => {
		if (
			refCreateName.value &&
			refCreateName.value !== "" &&
			refCreateName.value.length > 0 &&
			refCreateName.value.length <= 100 &&
			refCreatePermitsTotal.value >= refCreatePermitsUsed.value
		) {
			return true;
		}
		return false;
	});

	const factionOptions = computed<PSelectOption[]>(() => [
		{ label: t("empire.config.factions.NONE"), value: "NONE" },
		{ label: t("empire.config.factions.ANTARES"), value: "ANTARES" },
		{ label: t("empire.config.factions.BENTEN"), value: "BENTEN" },
		{ label: t("empire.config.factions.HORTUS"), value: "HORTUS" },
		{ label: t("empire.config.factions.MORIA"), value: "MORIA" },
		{ label: t("empire.config.factions.OUTSIDEREGION"), value: "OUTSIDEREGION" },
	]);

	generateEmpireCXMap();
	generateCXOptions();

	function generateEmpireCXMap(): void {
		const map: Record<string, string | undefined> = {};

		localEmpires.value.forEach((e) => {
			// identify which cx is used
			map[e.uuid] = undefined;

			localCX.value.forEach((c) => {
				if (c.empires.find((ce) => ce.uuid === e.uuid)) {
					map[e.uuid] = c.uuid;
				}
			});
		});

		refEmpireCXMap.value = map;
	}

	function generateCXOptions(): void {
		const options: PSelectOption[] = [
			{
				value: undefined,
				label: t("empire.plan_list.cogc_mapping.---"),
			},
		];

		localCX.value.forEach((c) => {
			options.push({
				value: c.uuid,
				label: c.name,
			});
		});

		refCXOptions.value = options;
	}

	const cxEmpireJunctions: ComputedRef<ICXEmpireJunction[]> = computed(() => {
		const jct: ICXEmpireJunction[] = [];

		// use all cx
		localCX.value.forEach((cx) => {
			const point: ICXEmpireJunction = {
				cx_uuid: cx.uuid,
				empires: [],
			};

			// find empires that have this setup
			Object.entries(refEmpireCXMap.value).forEach((entry) => {
				if (entry[1] === cx.uuid) {
					point.empires.push({
						empire_uuid: entry[0],
					});
				}
			});

			jct.push(point);
		});

		return jct;
	});

	async function updateCXJunctions(): Promise<void> {
		refIsUpdatingJunctions.value = true;

		try {
			trackEvent("manage_empire_junctions_update");

			await useQuery("PatchEmpireCXJunctions", {
				junctions: cxEmpireJunctions.value,
			}).execute();

			emit("update:cxList", await useQuery("GetAllCX").execute());
			refCreateName.value = "";
		} catch (err) {
			console.error(err);
		} finally {
			refIsUpdatingJunctions.value = false;
		}
	}

	async function createEmpire(): Promise<void> {
		refIsCreating.value = true;

		try {
			if (compCanCreate.value) {
				trackEvent("manage_empire_create");

				await useQuery("CreateEmpire", {
					data: {
						faction: refCreateFaction.value,
						permits_used: refCreatePermitsUsed.value,
						permits_total: refCreatePermitsTotal.value,
						name: refCreateName.value!,
						use_fio_storage: refCreateUseFioStorage.value,
					},
				}).execute();

				// forced reload of all Empires
				emit(
					"update:empireList",
					await useQuery("GetAllEmpires").execute()
				);
			}
		} catch (err) {
			console.error(err);
		} finally {
			refIsCreating.value = false;
			refShowCreateEmpire.value = false;
		}
	}

	function handleDeleteConfirm(empireUuid: string): void {
		dialog.warning({
			title: t("manage.empires.delete_confirm.title"),
			content: t("manage.empires.delete_confirm.content"),
			positiveText: t("manage.empires.delete_confirm.positive"),
			negativeText: t("manage.empires.delete_confirm.negative"),
			onPositiveClick: () => {
				deleteEmpire(empireUuid);
			},
		});
	}

	async function deleteEmpire(empireUuid: string): Promise<void> {
		refIsDeleting.value = empireUuid;
		trackEvent("manage_empire_delete", { empireUuid: empireUuid });
		const deletionResult: boolean = await useQuery("DeleteEmpire", {
			empireUuid: empireUuid,
		}).execute();

		if (deletionResult) {
			// forced reload of all Empires
			emit(
				"update:empireList",
				await useQuery("GetAllEmpires").execute()
			);
		}

		refIsDeleting.value = undefined;
	}
</script>

<template>
	<div class="flex flex-row flex-wrap gap-3 justify-between">
		<h2 class="text-xl font-bold my-auto">{{ $t("manage.empires.heading") }}</h2>
		<div class="flex gap-x-3">
			<PButton
				:loading="refIsUpdatingJunctions"
				@click="updateCXJunctions">
				<template #icon><SaveSharp /></template>
				{{ $t("manage.empires.update_cx_button") }}
			</PButton>
			<PButton @click="refShowCreateEmpire = !refShowCreateEmpire">
				<template #icon><PlusSharp /></template>
				{{ $t("manage.empires.new_empire_button") }}
			</PButton>
		</div>
	</div>
	<div class="py-3 text-white/60">
		{{ $t("manage.empires.info_text") }}
	</div>
	<div
		:class="
			!refShowCreateEmpire
				? 'opacity-0 overflow-hidden !h-0'
				: 'opacity-100'
		"
		class="transition-all duration-500 border-t border-b border-white/10">
		<div class="flex gap-x-3 pt-3 w-1/2 min-w-[400px]">
			<div class="flex-grow">
				<PForm>
					<PFormItem :label="$t('manage.empires.create_section.name_label')">
						<PInput
							v-model:value="refCreateName"
							class="w-full"
							:placeholder="$t('manage.empires.create_section.name_placeholder')" />
					</PFormItem>
					<PFormItem :label="$t('manage.empires.create_section.faction_label')">
						<PSelect
							v-model:value="refCreateFaction"
							class="w-full"
							:options="factionOptions" />
					</PFormItem>
					<PFormItem :label="$t('manage.empires.create_section.permits_total_label')">
						<PInputNumber
							v-model:value="refCreatePermitsTotal"
							show-buttons
							:min="2"
							class="w-full" />
					</PFormItem>
					<PFormItem :label="$t('manage.empires.create_section.permits_used_label')">
						<PInputNumber
							v-model:value="refCreatePermitsUsed"
							show-buttons
							:min="1"
							class="w-full" />
					</PFormItem>
					<PFormItem :label="$t('manage.empires.create_section.use_fio_label')">
						<PCheckbox v-model:checked="refCreateUseFioStorage" />
					</PFormItem>
				</PForm>
			</div>
			<div>
				<PButton
					:disabled="!compCanCreate"
					:loading="refIsCreating"
					@click="createEmpire">
					{{ $t("manage.empires.create_section.create_button") }}
				</PButton>
			</div>
		</div>
	</div>
	<x-n-data-table :data="localEmpires" striped class="pt-3">
		<x-n-data-table-column key="name" :title="$t('manage.empires.table.name')">
			<template #render-cell="{ rowData }">
				<router-link
					:to="`/empire/${rowData.uuid}`"
					class="text-link-primary font-bold hover:underline">
					{{ rowData.name }}
				</router-link>
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="faction" :title="$t('manage.empires.table.faction')">
			<template #render-cell="{ rowData }">
				{{ $t("empire.config.factions." + (rowData.faction || "NONE")) }}
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="permits" :title="$t('manage.empires.table.permits')">
			<template #render-cell="{ rowData }">
				{{ rowData.permits_used }} / {{ rowData.permits_total }}
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="plans" :title="$t('manage.empires.table.plans')">
			<template #render-cell="{ rowData }">
				{{ rowData.baseplanners.length }}
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="fio" :title="$t('manage.empires.table.fio')">
			<template #render-cell="{ rowData }">
				<PTag
					v-if="rowData.use_fio_storage"
					:bordered="false"
					type="success">
					{{ $t("manage.empires.table.yes") }}
				</PTag>
				<PTag v-else :bordered="false" type="error"> {{ $t("manage.empires.table.no") }} </PTag>
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="cx" :title="$t('manage.empires.table.cx')" width="200">
			<template #render-cell="{ rowData }">
				<div class="max-w-[200px]">
					<PSelect
						:key="`${rowData.uuid}#${refEmpireCXMap[rowData.uuid]}`"
						v-model:value="refEmpireCXMap[rowData.uuid]"
						:options="refCXOptions" />
				</div>
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="configuration" title="">
			<template #render-cell="{ rowData }">
				<div class="justify-end flex gap-x-3">
					<PButton
						size="sm"
						type="error"
						:loading="refIsDeleting === rowData.uuid"
						@click="handleDeleteConfirm(rowData.uuid)">
						<template #icon><ClearSharp /></template>
					</PButton>
				</div>
			</template>
		</x-n-data-table-column>
		<template #empty>
			<div class="flex flex-col gap-y-3">
				<div class="text-center">{{ $t("manage.empires.table.no_empires") }}</div>
				<div class="text-center">{{ $t("manage.empires.table.create_first") }}</div>
			</div>
		</template>
	</x-n-data-table>
</template>
