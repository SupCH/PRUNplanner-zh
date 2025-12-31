<script setup lang="ts">
	import { computed, ref, Ref } from "vue";

	// Composables
	import { useEndpoints } from "@/features/endpoints/useEndpoints";

	// Util
	import { relativeFromDate } from "@/util/date";

	// Types & Interfaces
	import { IUserAPIKey } from "@/features/api/userData.types";

	// UI
	import { PInput, PButton, PTable } from "@/ui";
	import { ClearSharp } from "@vicons/material";

	const { apiKeys } = defineProps<{ apiKeys: IUserAPIKey[] }>();

	const emit = defineEmits<{
		(e: "reload:keys"): void;
	}>();

	const { createAPIKey, deleteAPIKey } = useEndpoints();

	const localAPIKeys = computed(() =>
		[...apiKeys].sort((a, b) => (a.name > b.name ? 1 : -1))
	);

	const newAPIKeyName: Ref<string | null> = ref(null);

	const canCreate = computed(() =>
		apiKeys.length === 5 ||
		!newAPIKeyName.value ||
		newAPIKeyName.value === null ||
		newAPIKeyName.value === "" ||
		newAPIKeyName.value.length > 100
			? false
			: true
	);

	async function triggerKeyCreation() {
		if (!canCreate.value || newAPIKeyName.value === null) return;

		await createAPIKey(newAPIKeyName.value);
		emit("reload:keys");
	}

	async function triggerKeyDeletion(key: string) {
		await deleteAPIKey(key);
		emit("reload:keys");
	}
</script>

<template>
	<div class="flex flex-row flex-wrap justify-between child:my-auto pb-3">
		<h2 class="text-white/80 font-bold text-lg">{{ $t("api.management.title") }}</h2>
		<div class="flex flex-row flex-wrap gap-1">
			<div v-if="apiKeys.length >= 5" class="my-auto">
				{{ $t("api.management.limit_reached") }}
			</div>
			<template v-else>
				<PInput
					v-model:value="newAPIKeyName"
					:placeholder="$t('api.management.name_placeholder')" />
				<PButton :disabled="!canCreate" @click="triggerKeyCreation">
					{{ $t("api.management.create_button") }}
				</PButton>
			</template>
		</div>
	</div>

	<PTable striped>
		<thead>
			<tr>
				<th>{{ $t("api.management.table.name") }}</th>
				<th>{{ $t("api.management.table.key") }}</th>
				<th>{{ $t("api.management.table.created") }}</th>
				<th>{{ $t("api.management.table.last_activity") }}</th>
				<th></th>
			</tr>
		</thead>
		<tbody>
			<tr v-for="key in localAPIKeys" :key="key.key">
				<td>{{ key.name }}</td>
				<td class="font-mono text-nowrap">{{ key.key }}</td>
				<td class="text-nowrap">
					{{ relativeFromDate(key.created_date) }}
				</td>
				<td class="text-nowrap">
					{{
						key.last_activity
							? relativeFromDate(key.last_activity)
							: "&mdash;"
					}}
				</td>
				<td class="text-end">
					<PButton
						type="error"
						size="sm"
						@click="triggerKeyDeletion(key.key)">
						<template #icon>
							<ClearSharp />
						</template>
					</PButton>
				</td>
			</tr>
		</tbody>
	</PTable>
</template>
