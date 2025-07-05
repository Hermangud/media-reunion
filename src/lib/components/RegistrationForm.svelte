<script lang="ts">
	import { createClient, type SupabaseClient } from '@supabase/supabase-js';

	let { onSuccess }: { onSuccess: () => void } = $props();

	// Henter Supabase-nøklene sikkert fra miljøvariabler
	const supabaseUrl: string = import.meta.env.VITE_SUPABASE_URL;
	const supabaseKey: string = import.meta.env.VITE_SUPABASE_ANON_KEY;
	const supabase: SupabaseClient = createClient(supabaseUrl, supabaseKey);

	// Definerer state med TypeScript-typer
	let filesToUpload = $state<File[]>([]);
	let isLoading = $state<boolean>(false);
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

	// Håndterer innsending av skjemaet
	async function handleFormSubmit(event: SubmitEvent): Promise<void> {
		event.preventDefault();
		isLoading = true;
		feedbackMessage = '';

		const form = event.currentTarget as HTMLFormElement;
		const formData = new FormData(form);
		const name = formData.get('name') as string;
		const email = formData.get('email') as string;

		try {
			// STEG 1: Last opp filer FØRST, hvis de finnes
			if (filesToUpload.length > 0) {
				const uploadPromises = filesToUpload.map((file) => {
					// Laster opp til roten med et unikt navn
					const filePath = `${crypto.randomUUID()}-${file.name}`;
					return supabase.storage.from('media-reunion-pictures').upload(filePath, file);
				});

				const uploadResults = await Promise.all(uploadPromises);
				const uploadError = uploadResults.find((result) => result.error)?.error;
				if (uploadError) throw uploadError; // Hopper til catch ved feil
			}

			// STEG 2: KUN hvis opplasting var vellykket, registrer brukeren
			const { error: rpcError } = await supabase.rpc('register_guest', {
				guest_name: name,
				guest_email: email
			});
			if (rpcError) throw rpcError;

			// Alt var vellykket!
			messageType = 'success';
			feedbackMessage = 'Takk, din påmelding er mottatt!';
			if (onSuccess) onSuccess(); // Send signal til +page.svelte
		} catch (error: any) {
			messageType = 'error';
			feedbackMessage = 'En feil oppstod. Prøv igjen: ' + error.message;
			if (error.message.includes('duplicate key')) {
				feedbackMessage = 'Denne e-posten er allerede registrert.';
			}
			console.error('Feil i påmelding:', error);
		} finally {
			isLoading = false;
			form.reset();
			filesToUpload = [];
		}
	}
</script>

<form onsubmit={handleFormSubmit}>
	<div class="mb-3">
		<label for="name" class="form-label">Fullt navn</label>
		<input type="text" id="name" name="name" class="form-control" required disabled={isLoading} />
	</div>

	<div class="mb-3">
		<label for="email" class="form-label">E-postadresse</label>
		<input
			type="email"
			id="email"
			name="email"
			class="form-control"
			required
			disabled={isLoading}
		/>
	</div>

	<div class="mb-3">
		<label for="images" class="form-label">Last opp bilder (valgfritt)</label>
		<p class="form-text">
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
			disabled={isLoading}
			accept="image/*"
		/>
	</div>

	<div class="d-grid">
		<button type="submit" class="btn btn-primary btn-lg" disabled={isLoading}>
			{#if isLoading}
				<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
				Laster opp...
			{:else}
				Meld meg på
			{/if}
		</button>
	</div>
</form>

{#if feedbackMessage}
	<div
		class="alert mt-4 {messageType === 'success' ? 'alert-success' : 'alert-danger'}"
		role="alert"
	>
		{feedbackMessage}
	</div>
{/if}
