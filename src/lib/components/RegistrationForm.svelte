<script lang="ts">
	import { createClient, type SupabaseClient } from '@supabase/supabase-js';

	let { onSuccess }: { onSuccess: (kid: number) => void } = $props();

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

	// Håndterer hele innsendingsprosessen
	async function handleFormSubmit(event: SubmitEvent): Promise<void> {
		event.preventDefault();
		isLoading = true;
		feedbackMessage = '';

		const form = event.currentTarget as HTMLFormElement;
		const formData = new FormData(form);
		const name = formData.get('name') as string;
		const email = formData.get('email') as string;

		// ... din validering for om filer er valgt ...

		// STEG 1: Kall den nye database-funksjonen
		const { data: newKid, error: rpcError } = await supabase.rpc('register_guest_and_get_kid', {
			guest_name: name,
			guest_email: email
		});

		if (rpcError) {
			console.error('Feil ved kall av RPC-funksjon:', rpcError);
			messageType = 'error';
			// Sjekk om feilen er pga. duplikat e-post
			if (rpcError.message.includes('duplicate key')) {
				feedbackMessage = 'Denne e-posten er allerede registrert.';
			} else {
				feedbackMessage = 'En ukjent feil oppstod: ' + rpcError.message;
			}
			isLoading = false;
			return;
		}

		// STEG 2: Last opp filer (bruk newKid som mappenavn)
		for (const file of filesToUpload) {
			// Lager en unik filsti for å unngå kollisjoner,
			// og organiserer bilder under hver gjests unike ID.
			const filePath = `/${name + file.name}`;

			const { error: uploadError } = await supabase.storage
				.from('media-reunion-pictures') // Navnet på din bucket
				.upload(filePath, file);

			// Håndterer en eventuell feil under opplastingen
			if (uploadError) {
				console.error('Opplastingsfeil:', uploadError);
				messageType = 'error';
				feedbackMessage = `Feil ved opplasting av ${file.name}.`;
				isLoading = false;
				return; // Avbryter prosessen hvis én fil feiler
			}
		}

		// STEG 3: Alt var vellykket
		isLoading = false;
		messageType = 'success';
		feedbackMessage = 'Takk! Din påmelding er mottatt. 👍';
		form.reset();
		filesToUpload = [];

		if (onSuccess) {
			onSuccess(newKid); // Send den nye KID-en til +page.svelte
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
