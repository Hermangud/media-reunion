<script lang="ts">
	let { eventName, location, date }: { eventName: string; location: string; date: string } =
		$props();
</script>

<header class="text-center py-4 mb-4 border-bottom">
	<h1 class="display-4">{eventName}</h1>
	<p class="lead text-muted">
		📍 {location} &nbsp;&bull;&nbsp; 🗓️ {date}
	</p>
</header>
