<script lang="ts">
	let { eventName, location, date }: { eventName: string; location: string; date: string } =
		$props();
</script>

<header class="text-center py-4 mb-4 border-bottom">
	<h1 class="display-4">{eventName}</h1>
	<p class="lead text-muted">
		📍
		<a
			href={`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(location)}`}
			target="_blank"
			rel="noopener noreferrer"
		>
			{location}
		</a>
		&nbsp;&bull;&nbsp; 🗓️ {date}
	</p>
</header>
