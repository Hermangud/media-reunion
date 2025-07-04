<script lang="ts">
	// All din eksisterende logikk forblir den samme
	import { createClient, type SupabaseClient } from '@supabase/supabase-js';

	const supabaseUrl: string = import.meta.env.VITE_SUPABASE_URL;
	const supabaseKey: string = import.meta.env.VITE_SUPABASE_ANON_KEY;
	const supabase: SupabaseClient = createClient(supabaseUrl, supabaseKey);

	let emailToSearch = $state<string>('');
	let isLoading = $state<boolean>(false);
	let searchStatus = $state<'idle' | 'found' | 'not_found'>('idle');
	let errorMessage = $state<string>('');
	let paidStatus = $state<boolean | null>(null);

	async function handleStatusCheck(): Promise<void> {
		if (!emailToSearch) return;

		isLoading = true;
		searchStatus = 'idle';
		errorMessage = '';
		paidStatus = null; // Nullstill betalingsstatus ved nytt søk

		try {
			const { data, error } = await supabase
				.from('public_guest_status') // Spør mot det sikre viewet
				.select('paid') // Henter betalingsstatus
				.eq('email', emailToSearch)
				.single();

			if (error) {
				searchStatus = 'not_found';
			} else {
				searchStatus = 'found';
				// NY: Lagrer betalingsstatus fra svaret
				paidStatus = data.paid;
			}
		} catch (error: any) {
			errorMessage = 'En teknisk feil oppstod. Prøv igjen.';
			console.error('Uventet feil:', error);
		} finally {
			isLoading = false;
		}
	}
</script>

<div class="status-checker-container">
	<form onsubmit={handleStatusCheck}>
		<label for="status-email" class="form-label">Sjekk om din e-post er registrert</label>
		<div class="input-group">
			<input
				id="status-email"
				type="email"
				class="form-control"
				bind:value={emailToSearch}
				placeholder="din.epost@eksempel.com"
				required
				disabled={isLoading}
			/>
			<button class="btn btn-secondary" type="submit" disabled={isLoading}>
				{#if isLoading}
					<span class="spinner-border spinner-border-sm" aria-hidden="true"></span>
				{:else}
					Sjekk status
				{/if}
			</button>
		</div>
	</form>

	{#if isLoading}
		<div class="text-center mt-3">
			<div class="spinner-border text-secondary" role="status">
				<span class="visually-hidden">Laster...</span>
			</div>
		</div>
	{/if}

	{#if errorMessage}
		<div class="alert alert-danger mt-3">{errorMessage}</div>
	{/if}

	{#if searchStatus === 'found'}
		{#if paidStatus === true}
			<div class="alert alert-success mt-3">
				✅ Ja, din påmelding er registrert og betalt. Vi gleder oss til å se deg!
			</div>
		{:else}
			<div class="alert alert-warning mt-3">
				⚠️ Ja, vi har mottatt din påmelding, men mangler registrert betaling.
			</div>
		{/if}
	{/if}

	{#if searchStatus === 'not_found'}
		<div class="alert alert-danger mt-3">
			❌ Nei, vi finner ingen påmelding med denne e-postadressen.
		</div>
	{/if}
</div>
