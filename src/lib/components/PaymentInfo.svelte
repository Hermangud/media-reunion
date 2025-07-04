<script lang="ts">
	let { kidNumber }: { kidNumber: number | null } = $props();
</script>

<div class="row g-4">
	<div class="col-md-6">
		<div class="card h-100">
			<div class="card-body">
				<h3 class="card-title">Vipps</h3>
				<p class="card-text">
					Send <strong>250 kr</strong> kr til <strong>978 18 141</strong>
				</p>
				<p class="card-text text-muted small">
					Merk betalingen med navnet du brukte i påmeldingen.
				</p>
			</div>
		</div>
	</div>

	<div class="col-md-6">
		<div class="card h-100">
			<div class="card-body">
				<h3 class="card-title">Bankoverføring</h3>
				<p class="card-text">Kontonummer: <strong>4214 47 45 374</strong></p>
				<p class="card-text">
					KID: <strong>{kidNumber}</strong>
				</p>
			</div>
		</div>
	</div>
</div>
