<script lang="ts">
	let {
		x1,
		y1,
		x2,
		y2,
		color,
		arrowId,
		label = '',
		labelOffset = { x: 20, y: 10 },
		showWave = true,
		wavelengthNm = 500,
		refractiveIndex = 1, // default to air's refractive index
		nmToPixels = 0.3, // scale factor to convert nm to pixels
		waveStart = 'start' // 'start' or 'end' - determines where the wave's phase starts from
	} = $props();

	// Sine wave parameters
	const amplitude = 5; // Wave height in pixels

	// Calculate the angle and length for the sine wave
	const angle = $derived(Math.atan2(y2 - y1, x2 - x1));
	const length = $derived(Math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2));

	// Calculate wavelength in pixels, accounting for refractive index
	const wavelengthPx = $derived((wavelengthNm / refractiveIndex) * nmToPixels);

	// Generate sine wave path
	const wavePath = $derived(generateWavePath());

	function generateWavePath() {
		const numPoints = Math.ceil(length / 5); // One point every 5 pixels
		const points = [];

		for (let i = 0; i <= numPoints; i++) {
			const t = i / numPoints;
			const tPhase = waveStart === 'end' ? 1 - t : t;

			// Calculate position along the line
			const x = x1 + (x2 - x1) * t;
			const y = y1 + (y2 - y1) * t;

			// Calculate perpendicular offset for sine wave
			// Flip the phase by negating the sine when waveStart is 'end'
			const phaseMultiplier = waveStart === 'end' ? -1 : 1;
			const waveOffset =
				phaseMultiplier * Math.sin((tPhase * length * 2 * Math.PI) / wavelengthPx) * amplitude;
			const perpX = -Math.sin(angle) * waveOffset;
			const perpY = Math.cos(angle) * waveOffset;

			points.push(`${x + perpX},${y + perpY}`);
		}

		return `M ${points.join(' L ')}`;
	}
</script>

<g class="light-beam">
	<!-- Main arrow line -->
	<line
		{x1}
		{y1}
		{x2}
		{y2}
		stroke={color}
		stroke-width="1"
		marker-end="url(#{arrowId})"
		opacity=".3"
	/>

	<!-- Sine wave overlay -->
	{#if showWave}
		<path d={wavePath} stroke={color} stroke-width="2" fill="none" opacity="1" />
	{/if}

	<!-- Label -->
	{#if label}
		<text x={x2 + labelOffset.x} y={y2 + labelOffset.y} class="beam-label">{label}</text>
	{/if}
</g>

<style>
	.light-beam {
		pointer-events: none;
	}

	.beam-label {
		font-size: 12px;
		font-weight: 500;
		fill: #333;
	}
</style>
