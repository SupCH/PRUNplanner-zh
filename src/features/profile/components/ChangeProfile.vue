<script setup lang="ts">
	import { reactive, watch, Ref, ref, onMounted } from "vue";

	// API
	import { useQuery } from "@/lib/query_cache/useQuery";

	// Composables
	import { trackEvent } from "@/lib/analytics/useAnalytics";

	// Stores
	import { useUserStore } from "@/stores/userStore";

	// UI
	import { PForm, PFormItem, PInput, PButton, PCheckbox } from "@/ui";

	const userStore = useUserStore();

	// local profile copy
	const localProfile = reactive({ ...userStore.profile });
	const isUpdating: Ref<boolean> = ref(false);
	const wasSaved: Ref<boolean> = ref(true);
	const codeResendRequested: Ref<boolean> = ref(false);

	watch(
		() => userStore.profile,
		(newProfile) => Object.assign(localProfile, newProfile)
	);

	watch(
		() => localProfile,
		() => (wasSaved.value = false),
		{ deep: true }
	);

	onMounted(() =>
		userStore.performGetProfile().then(() => (wasSaved.value = true))
	);

	async function patchProfile(): Promise<void> {
		trackEvent("user_profile_change");

		isUpdating.value = true;

		try {
			await useQuery("PatchUserProfile", {
				fio_apikey: localProfile.fio_apikey?.replace(/ /g, "") ?? null,
				prun_username: localProfile.prun_username ?? null,
				email: localProfile.email ?? null,
			}).execute();

			wasSaved.value = true;
		} catch (err) {
			console.error("Error patching user profile", err);
		} finally {
			isUpdating.value = false;
		}
	}

	async function requestVerification(): Promise<void> {
		trackEvent("user_request_email_verification");

		try {
			await useQuery("PostUserResendEmailVerification", null).execute();
		} catch (err) {
			console.error("Error resending verification code", err);
		} finally {
			codeResendRequested.value = true;
		}
	}
</script>

<template>
	<div>
		<div class="flex flex-row flex-wrap gap-3">
			<h2 class="flex-grow my-auto text-white/80 font-bold text-lg">
				{{ $t("profile.change_profile.title") }}
			</h2>
			<PButton
				:loading="isUpdating"
				:type="wasSaved ? 'primary' : 'error'"
				@click="patchProfile">
				{{ $t("profile.change_profile.update_button") }}
			</PButton>
		</div>
		<div class="py-3 text-white/60">
			{{ $t("profile.change_profile.description") }}
		</div>
		<PForm v-if="localProfile">
			<PFormItem :label="$t('profile.change_profile.fio_key_label')">
				<PInput
					v-model:value="localProfile.fio_apikey"
					class="w-full min-w-[200px] max-w-[50%]" />
			</PFormItem>
			<PFormItem :label="$t('profile.change_profile.username_label')">
				<PInput
					v-model:value="localProfile.prun_username"
					class="w-full min-w-[200px] max-w-[50%]" />
			</PFormItem>
			<PFormItem :label="$t('profile.change_profile.email_label')">
				<PInput
					v-model:value="localProfile.email"
					class="w-full min-w-[200px] max-w-[50%]" />
			</PFormItem>
			<PFormItem :label="$t('profile.change_profile.email_verified_label')">
				<div class="w-full flex flex-row flex-wrap gap-3">
					<PCheckbox
						v-model:checked="localProfile.email_verified"
						disabled
						class="w-full min-w-[200px] max-w-[50%] child:my-auto" />

					<div
						v-if="!localProfile.email_verified"
						class="flex flex-row flex-wrap gap-3">
						<div>
							<router-link
								to="/verify-email"
								class="text-link-primary hover:cursor-pointer hover:underline">
								{{ $t("profile.change_profile.verify_email_button") }}
							</router-link>
						</div>
						<div>
							<span
								v-if="!codeResendRequested"
								class="text-link-primary hover:cursor-pointer hover:underline"
								@click="requestVerification">
								{{ $t("profile.change_profile.resend_code_button") }}
							</span>
							<span v-else class="text-lime-600">
								{{ $t("profile.change_profile.code_requested") }}
							</span>
						</div>
					</div>
				</div>
			</PFormItem>
		</PForm>
	</div>
</template>
