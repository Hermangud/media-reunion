<script lang="ts">
	// All din eksisterende logikk forblir den samme
	import { createClient, type SupabaseClient } from '@supabase/supabase-js';

	const supabaseUrl: string = import.meta.env.VITE_SUPABASE_URL;
	const supabaseKey: string = import.meta.env.VITE_SUPABASE_ANON_KEY;
	const supabase: SupabaseClient = createClient(supabaseUrl, supabaseKey);

	let filesToUpload = $state<File[]>([]);
	let isLoading = $state<boolean>(false);
	let feedbackMessage = $state<string>('');
	let messageType = $state<'success' | 'error'>('success');

	function handleFileChange(event: Event) {
		const input = event.currentTarget as HTMLInputElement;
		if (input.files) {
			filesToUpload = [...input.files];
		}
	}

	async function handleFormSubmit(event: SubmitEvent): Promise<void> {
		// Hele din handleFormSubmit-funksjon forblir uendret
	}
</script>

<form on:submit={handleFormSubmit}>
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
		<label for="images" class="form-label">Last opp bilder</label>
		<input
			type="file"
			id="images"
			name="images"
			multiple
			class="form-control"
			on:change={handleFileChange}
			disabled={isLoading}
			accept="image/*"
		/>
		<div class="form-text">Du kan laste opp flere bilder samtidig.</div>
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
