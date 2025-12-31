<script setup lang="ts">
	import { onMounted, PropType, ref, Ref } from "vue";

	// Composables
	import { useSharing } from "@/features/sharing/useSharing";
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Util
	import { copyToClipboard } from "@/util/data";

	// UI
	import { NModal } from "naive-ui";
	import { PButton } from "@/ui";
	import { LinkSharp, RemoveRedEyeSharp } from "@vicons/material";

	const props = defineProps({
		planUuid: {
			type: String,
			required: true,
		},
		buttonSize: {
			type: String as PropType<"sm" | "md">,
			required: false,
			default: "md",
		},
		load: {
			type: Boolean,
			required: false,
			default: false,
		},
	});

	const {
		isShared,
		viewCount,
		url,
		deleteSharing,
		createSharing,
		refreshStore,
	} = useSharing(props.planUuid);

	const showModal: Ref<boolean> = ref(false);
	const isDeleting: Ref<boolean> = ref(false);
	const isCreating: Ref<boolean> = ref(false);

	async function stopSharing(): Promise<void> {
		isDeleting.value = true;
		await deleteSharing();
		isDeleting.value = false;
		showModal.value = false;
		trackEvent("plan_share_delete");
	}

	async function doCreateSharing(): Promise<void> {
		isCreating.value = true;
		await createSharing();
		trackEvent("plan_share_create");
		isCreating.value = false;
	}

	onMounted(() => {
		if (props.load) {
			refreshStore();
		}
	});
</script>

<template>
	<n-modal
		v-model:show="showModal"
		class="!w-fit !max-w-[700px]"
		preset="card"
		:title="$t('sharing.title')">
		<template v-if="!isShared">
			<div>
				{{ $t("sharing.create_info") }}
			</div>
		</template>
		<template v-else>
			<div class="pb-3">
				{{ $t("sharing.active_info", { count: viewCount }) }}
			</div>
			<div v-if="url" class="font-mono">
				{{ url }}
			</div>
		</template>
		<template v-if="!isShared" #action>
			<PButton
				:size="buttonSize"
				:loading="isCreating"
				@click="doCreateSharing">
				{{ $t("sharing.create_link_button") }}
			</PButton>
		</template>
		<template v-else #action>
			<div class="flex justify-between">
				<PButton
					v-if="url"
					:size="buttonSize"
					type="success"
					@click="copyToClipboard(url)">
					{{ $t("sharing.copy_link_button") }}
				</PButton>
				<PButton
					:size="buttonSize"
					type="error"
					:loading="isDeleting"
					@click="stopSharing">
					{{ $t("sharing.stop_sharing_button") }}
				</PButton>
			</div>
		</template>
	</n-modal>

	<PButton
		:size="buttonSize"
		:type="isShared ? 'success' : 'primary'"
		@click="() => (showModal = !showModal)">
		<template v-if="isShared" #icon><RemoveRedEyeSharp /></template>
		<template v-else #icon><LinkSharp /></template>

		<template v-if="isShared">{{ viewCount }}</template>

		<template v-if="buttonSize !== 'sm'">
			<template v-if="isShared"> {{ $t("sharing.views_label") }}</template>
			<template v-else>{{ $t("sharing.share_label") }}</template>
		</template>
	</PButton>
</template>
