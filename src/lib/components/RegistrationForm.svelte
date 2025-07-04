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

		// Sjekker om brukeren har lastet opp filer
		if (filesToUpload.length > 0) {
			// ----- LOGIKK FOR PÅMELDING MED BILDER -----

			const uploadedFilePaths: string[] = [];
			try {
				const uploadPromises = filesToUpload.map((file) => {
					const filePath = `temp/${crypto.randomUUID()}-${file.name}`;
					uploadedFilePaths.push(filePath);
					return supabase.storage.from('media-reunion-pictures').upload(filePath, file);
				});

				const uploadResults = await Promise.all(uploadPromises);
				const firstError = uploadResults.find((result) => result.error);
				if (firstError) throw firstError.error;
			} catch (error) {
				// Opprydding ved feil
				if (uploadedFilePaths.length > 0) {
					await supabase.storage.from('media-reunion-pictures').remove(uploadedFilePaths);
				}
				messageType = 'error';
				feedbackMessage = 'En feil oppstod under filopplasting. Prøv igjen.';
				isLoading = false;
				return;
			}

			const { data: newId, error: rpcError } = await supabase.rpc('register_guest_and_get_kid', {
				guest_name: name,
				guest_email: email
			});

			if (rpcError) {
				// Opprydding ved databasefeil
				await supabase.storage.from('media-reunion-pictures').remove(uploadedFilePaths);
				// ... din eksisterende feilhåndtering ...
				isLoading = false;
				return;
			}

			// Flytt filer fra temp til endelig mappe
			for (const tempPath of uploadedFilePaths) {
				const fileName = tempPath.split('/')[1];
				const newPath = `${newId}/${fileName}`;
				await supabase.storage.from('media-reunion-pictures').move(tempPath, newPath);
			}

			// Suksessmelding for påmelding MED bilder
			messageType = 'success';
			feedbackMessage = 'Takk! Din påmelding og bilder er mottatt. 👍';
			if (onSuccess) onSuccess(newId);
		} else {
			// ----- LOGIKK FOR PÅMELDING UTEN BILDER -----

			const { data: newId, error: rpcError } = await supabase.rpc('register_guest_and_get_kid', {
				guest_name: name,
				guest_email: email
			});

			if (rpcError) {
				// Standard feilhåndtering for duplikat e-post etc.
				messageType = 'error';
				if (rpcError.message.includes('duplicate key')) {
					feedbackMessage = 'Denne e-posten er allerede registrert.';
				} else {
					feedbackMessage = 'En ukjent databasefeil oppstod.';
				}
				isLoading = false;
				return;
			}

			// UNIK suksessmelding for påmelding UTEN bilder
			messageType = 'success';
			feedbackMessage = 'Takk for din påmelding! Du lastet ikke opp noen bilder.';
			if (onSuccess) onSuccess(newId);
		}

		// Felles opprydding til slutt
		isLoading = false;
		form.reset();
		filesToUpload = [];
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
