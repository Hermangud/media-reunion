<script lang="ts">
	import { supabase } from '$lib/supabaseClient';
	let filesToUpload = $state<File[]>([]);
	let uploading = $state<boolean>(false);
	let feedbackMessage = $state<string>('');
	let messageType = $state<'success' | 'error'>('success');

	// Håndterer filvalg fra input-feltet
	function handleFileChange(event: Event) {
		const input = event.currentTarget as HTMLInputElement;
		if (input.files) {
			// Konverterer den spesielle FileList-typen til en vanlig File[] array
			filesToUpload = [...input.files];
		}
	}

	async function handleUpload() {
		if (filesToUpload.length > 0) {
			uploading = true;
			const uploadPromises = filesToUpload.map((file) => {
				const filePath = `${crypto.randomUUID()}-${file.name}`;
				return supabase.storage.from('media-reunion-pictures').upload(filePath, file);
			});
			const uploadResults = await Promise.all(uploadPromises);
			const uploadError = uploadResults.find((result) => result.error)?.error;
			uploading = false;
			if (uploadError) {
				feedbackMessage = 'Noe gikk galt, prøv igjen: ' + uploadError.message;
				messageType = 'error';
				throw uploadError;
			}
			feedbackMessage.length > 0
				? (feedbackMessage = 'Bildene ble lastet opp!')
				: (feedbackMessage = 'Bildet ble lastet opp');
			messageType = 'success';
			// Nullstill input-feltet etter opplasting
			const input = document.getElementById('images') as HTMLInputElement;
			if (input) input.value = '';
			// Nullstill filene etter opplasting
			filesToUpload = [];
		}
	}
</script>

<section id="image-upload" class="mb-5">
	<h2 class="mb-3">Last opp bilder</h2>
	<p class="text-muted mb-3">
		Bildene vil bli brukt til bildekarusell under kvelden. Del dine beste minnner fra tiden vår på
		Glemmen!
	</p>
	<input
		type="file"
		id="images"
		name="images"
		multiple
		class="form-control"
		onchange={handleFileChange}
		disabled={uploading}
		accept="image/*"
	/>

	<div class="d-grid mt-3">
		<button
			type="button"
			class="btn btn-primary btn-lg"
			onclick={handleUpload}
			disabled={uploading || filesToUpload.length === 0}
		>
			{#if uploading}
				<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
				Laster opp...
			{:else}
				Last opp bilder
			{/if}
		</button>
	</div>

	{#if feedbackMessage}
		<div
			class="alert mt-4 {messageType === 'success' ? 'alert-success' : 'alert-danger'}"
			role="alert"
		>
			{feedbackMessage}
		</div>
	{/if}
</section>
