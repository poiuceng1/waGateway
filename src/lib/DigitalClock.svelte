<!-- src/DigitalClock.svelte -->
<script>
	let time = new Date();

	function updateTime() {
		time = new Date();
	}

	setInterval(updateTime, 1000);
	updateTime();
</script>

<div class="clock">
	{time.toLocaleTimeString()}
</div>

<style>
	.clock {
		font-size: 2rem;
		font-weight: bold;
		color: #333;
	}
</style>
