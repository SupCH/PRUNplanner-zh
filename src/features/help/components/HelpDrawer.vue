<script setup lang="ts">
	import { PropType, ref, Ref, watch } from "vue";
	import { useI18n } from "vue-i18n";
	const { t, locale } = useI18n();

	import { PButton } from "@/ui";
	import { NDrawer, NDrawerContent } from "naive-ui";
	import { VueShowdown } from "vue-showdown";

	const showDrawer: Ref<boolean> = ref(false);

	async function loadMarkdown(): Promise<string> {
		const markdownFiles = import.meta.glob("@/assets/help/*.md", {
			query: "?raw",
			import: "default",
		}) as Record<string, () => Promise<string>>;

		const localeSuffix = locale.value === "zh" ? ".zh" : "";
		let path = `/src/assets/help/${props.fileName}${localeSuffix}.md`;

		if (!markdownFiles[path]) {
			path = `/src/assets/help/${props.fileName}.md`;
		}

		const loader = markdownFiles[path];
		if (!loader)
			throw new Error(`Markdown file "${props.fileName}" not found.`);
		return await loader();
	}

	const markdownContent = ref("");

	const props = defineProps({
		fileName: {
			type: String,
			required: true,
		},
		drawerTitle: {
			type: String,
			required: false,
			default: undefined,
		},
		buttonTitle: {
			type: String,
			required: false,
			default: undefined,
		},
		buttonSize: {
			type: String as PropType<"sm" | "md">,
			required: false,
			default: "md",
		},
		buttonClass: {
			type: String,
			required: false,
			default: "",
		},
		drawerWidth: {
			type: Number,
			required: false,
			default: 600,
		},
	});

	watch([showDrawer, locale], async () => {
		if (showDrawer.value) {
			markdownContent.value = await loadMarkdown();
		}
	});
</script>

<template>
	<PButton
		:size="buttonSize"
		:class="buttonClass"
		type="secondary"
		@click="() => (showDrawer = !showDrawer)">
		{{ buttonTitle ? buttonTitle : $t("help.title") }}
	</PButton>
	<n-drawer v-model:show="showDrawer" :width="drawerWidth" placement="right">
		<n-drawer-content closable>
			<template #header>
				{{ drawerTitle ? drawerTitle : $t("help.title") }}
			</template>
			<div v-if="markdownContent != ''" id="markdown">
				<VueShowdown :markdown="markdownContent" />
			</div>
			<div v-else class="text-center text-red-500">
				{{ $t("help.unable_to_load", { file: fileName }) }}
			</div>
		</n-drawer-content>
	</n-drawer>
</template>
