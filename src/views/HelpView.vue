<script setup lang="ts">
	import { onMounted, Ref, ref } from "vue";
	import { useHead } from "@unhead/vue";
	import { useI18n } from "vue-i18n";
	const { t, locale } = useI18n();

	useHead({
		title: `${t("help.title")} | PRUNplanner`,
	});

	import HelpTutorial from "@/features/help/components/HelpTutorial.vue";
	import { VueShowdown } from "vue-showdown";
	import { watch } from "vue";

	// markdown loader from changelog
	async function loadMarkdown(): Promise<string> {
		const markdownFiles = import.meta.glob("@/assets/help/*.md", {
			query: "?raw",
			import: "default",
		}) as Record<string, () => Promise<string>>;

		const path = `/src/assets/help/changelog${
			locale.value === "zh" ? ".zh" : ""
		}.md`;
		const loader = markdownFiles[path];
		if (!loader) throw new Error(`Markdown file "changelog" not found.`);

		return await loader();
	}

	const markdownContent: Ref<string> = ref("");

	onMounted(async () => (markdownContent.value = await loadMarkdown()));

	watch(locale, async () => {
		markdownContent.value = await loadMarkdown();
	});
</script>

<template>
	<div class="min-h-screen flex flex-col">
		<div
			class="px-6 py-3 border-b border-white/10 flex flex-row justify-between">
			<h1 class="text-2xl font-bold my-auto">{{ $t("help.heading") }}</h1>
		</div>

		<div
			class="flex-grow grid grid-cols-1 lg:grid-cols-[60%_auto] gap-3 divide-x divide-white/10 child:px-6 child:py-3 child:last:pl-3">
			<div>
				<HelpTutorial />
			</div>
			<div>
				<section class="bg-white/10 p-3 rounded mb-3 flex flex-col gap-2">
					<div>
						{{ $t("help.contribution_text") }}
						<a
							href="https://github.com/PRUNplanner/frontend"
							target="_blank"
							class="text-link-primary font-bold hover:underline"
							>PRUNplanner</a
						>
					</div>
					<div class="text-sm border-t border-white/5 pt-2">
						<span class="opacity-60">中文版项目：</span>
						<a
							href="https://github.com/SupCH/PRUNplanner-zh"
							target="_blank"
							class="text-link-primary font-bold hover:underline"
							>PRUNplanner-zh</a
						>
					</div>
				</section>

				<h2 class="text-xl font-bold pb-3">{{ $t("help.changelog") }}</h2>
				<div
					v-if="markdownContent != ''"
					id="markdown"
					class="h-screen overflow-auto">
					<VueShowdown :markdown="markdownContent" />
				</div>
			</div>
		</div>
	</div>
</template>
