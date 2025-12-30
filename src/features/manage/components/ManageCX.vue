<script setup lang="ts">
	import { computed, ComputedRef, PropType, ref, Ref } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	// Composables
	import { useQuery } from "@/lib/query_cache/useQuery";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Types & Interfaces
	import { ICX } from "@/stores/planningStore.types";

	// Util
	import { inertClone } from "@/util/data";

	// UI
	import { PButton, PInput, PTag } from "@/ui";
	import { useDialog } from "naive-ui";
	const dialog = useDialog();
	import { XNDataTable, XNDataTableColumn } from "@skit/x.naive-ui";
	import { ClearSharp, PlusSharp } from "@vicons/material";

	const props = defineProps({
		cx: {
			type: Array as PropType<ICX[]>,
			required: true,
		},
	});

	// Local Data
	const localCX: ComputedRef<ICX[]> = computed(() => inertClone(props.cx));

	const refShowCreateCX: Ref<boolean> = ref(false);
	const refIsCreating: Ref<boolean> = ref(false);
	const refIsDeleting: Ref<string | undefined> = ref(undefined);
	const refNewCXName: Ref<string | undefined> = ref(undefined);
	const compCanCreate: ComputedRef<boolean> = computed(() => {
		if (
			refNewCXName.value &&
			refNewCXName.value !== "" &&
			refNewCXName.value.length > 0 &&
			refNewCXName.value.length <= 100
		)
			return true;
		return false;
	});

	const emit = defineEmits<{
		(e: "update:cxList", value: ICX[]): void;
	}>();

	async function createCX(): Promise<void> {
		if (compCanCreate.value) {
			refIsCreating.value = true;

			trackEvent("manage_cx_create");

			await useQuery("CreateCX", {
				cxName: refNewCXName.value!,
			}).execute();

			// forced reload of all CX
			emit("update:cxList", await useQuery("GetAllCX").execute());

			refNewCXName.value = "";
			refShowCreateCX.value = false;
			refIsCreating.value = false;
		}
	}

	function handleDeleteConfirm(cxUuid: string): void {
		dialog.warning({
			title: t("manage.exchanges.delete_confirm.title"),
			content: t("manage.exchanges.delete_confirm.content"),
			positiveText: t("manage.exchanges.delete_confirm.positive"),
			negativeText: t("manage.exchanges.delete_confirm.negative"),
			onPositiveClick: () => {
				deleteCX(cxUuid);
			},
		});
	}

	async function deleteCX(cxUuid: string): Promise<void> {
		refIsDeleting.value = cxUuid;

		trackEvent("manage_cx_delete", { cxUuid });

		const deletionResult: boolean = await useQuery("DeleteCX", {
			cxUuid: cxUuid,
		}).execute();

		if (deletionResult) {
			// forced reload of all CX
			emit("update:cxList", await useQuery("GetAllCX").execute());
		}

		refIsDeleting.value = undefined;
	}
</script>

<template>
	<div class="flex flex-row flex-wrap gap-3 justify-between">
		<h2 class="text-xl font-bold my-auto">{{ $t("manage.exchanges.heading") }}</h2>
		<PButton @click="refShowCreateCX = !refShowCreateCX">
			<template #icon><PlusSharp /></template>
			{{ $t("manage.exchanges.new_cx_button") }}
		</PButton>
	</div>
	<div class="py-3 text-white/60">
		{{ $t("manage.exchanges.info_text") }}
	</div>
	<div
		:class="
			!refShowCreateCX ? 'opacity-0 overflow-hidden !h-0' : 'opacity-100'
		"
		class="transition-all duration-500 border-t border-b border-white/10">
		<div class="flex gap-x-3 py-2">
			<div class="my-auto">{{ $t("manage.exchanges.create_section.name_label") }}</div>
			<div class="flex-grow">
				<PInput
					v-model:value="refNewCXName"
					:placeholder="$t('manage.exchanges.create_section.name_placeholder')" />
			</div>
			<PButton
				:loading="refIsCreating"
				:disabled="!compCanCreate"
				@click="createCX">
				{{ $t("manage.exchanges.create_section.create_button") }}
			</PButton>
		</div>
	</div>

	<x-n-data-table :data="localCX" striped class="pt-3">
		<x-n-data-table-column key="name" :title="$t('manage.exchanges.table.name')">
			<template #render-cell="{ rowData }">
				<router-link
					:to="`/exchanges/${rowData.uuid}`"
					class="text-link-primary font-bold hover:underline">
					{{ rowData.name }}
				</router-link>
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="uuid" :title="$t('manage.exchanges.table.assigned')">
			<template #title>
				<div class="text-center">{{ $t("manage.exchanges.table.assigned") }}</div>
			</template>
			<template #render-cell="{ rowData }">
				<div class="text-center">
					<PTag v-if="rowData.empires.length > 0" type="success">
						{{ $t("manage.exchanges.table.yes") }} ({{ rowData.empires.length }})
					</PTag>
					<PTag v-else type="error"> {{ $t("manage.exchanges.table.no") }} </PTag>
				</div>
			</template>
		</x-n-data-table-column>
		<x-n-data-table-column key="configuration" title="">
			<template #render-cell="{ rowData }">
				<div class="text-end">
					<PButton
						size="sm"
						type="error"
						:loading="refIsDeleting === rowData.uuid ? true : false"
						@click="handleDeleteConfirm(rowData.uuid)">
						<template #icon><ClearSharp /></template>
					</PButton>
				</div>
			</template>
		</x-n-data-table-column>
		<template #empty>
			<div class="flex flex-col gap-y-3">
				<div class="text-center">{{ $t("manage.exchanges.table.no_cx") }}</div>
				<div class="text-center">
					{{ $t("manage.exchanges.table.create_first") }}
				</div>
			</div>
		</template>
	</x-n-data-table>
</template>
