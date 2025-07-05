<script lang="ts">
	import qrVipps from '$lib/assets/qr-vipps.svg';
	let { kidNumber }: { kidNumber: number | null } = $props();
</script>

<div class="row g-4">
	<div class="col-md-6">
		<div class="card h-100">
			<div class="card-body mx-a">
				<h3 class="card-title">Vipps</h3>
				<p class="card-text">
					Send <strong>250 kr</strong> kr til <strong>978 18 141</strong>
				</p>
				<img class="w-50" src={qrVipps} alt="QR kode til Vipps" />
			</div>
		</div>
	</div>

	<div class="col-md-6">
		<div class="card h-100">
			<div class="card-body">
				<h3 class="card-title">Bankoverføring</h3>
				<p class="card-text">Beløp: <strong>250 kr</strong></p>
				<p class="card-text">Kontonummer: <strong>4214 47 45374</strong></p>
				<p class="card-text">
					KID: <strong>{kidNumber}</strong>
				</p>
			</div>
		</div>
	</div>
</div>
