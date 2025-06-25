<script lang="ts">
	//
	// Physics simulation of a single ray of light hitting a thin film on a
	// substrate, used to demonstrate the constructive and destructive interference
	// for introductory-level physics classes
	//

	import LightBeam from './LightBeam.svelte';

	// Non-adjustable parameters
	const wavelengthNm = 500;
	const airN = 1;
	const substrateN = 1.5; // index of refraction

	// Scale factor to convert nanometers to pixels
	const nmToPixels = 0.3; // This was the factor we were using for film thickness

	// User adjustable parameters
	let filmThicknessNm = $state(300);
	const filmThicknessRange = { min: 10, max: 700 };

	const filmMaterials = [
		{ label: 'Test', n: 3 },
		{ label: 'MgF2', n: 1.38 },
		{ label: 'SiO2', n: 1.65 }
	];
	let selectedFilmIndex = $state(0);

	let angleDegrees = $state(15); // angle from vertical
	const angleRange = { min: 0, max: 30 };

	// SVG dimensions and layout
	const svgWidth = 800;
	const svgHeight = 600;
	const centerX = svgWidth / 2;

	// Layer positions and dimensions
	const filmTop = 200;
	const filmBottom = $derived(filmTop + filmThicknessNm * nmToPixels);
	const substrateTop = $derived(filmBottom);
	const substrateBottom = 400;

	// Beam parameters
	const beamLength = 150;
	const beamStartY = 50;

	// Reactive calculations for beam paths
	const angleRad = $derived((angleDegrees * Math.PI) / 180);
	const filmN = $derived(filmMaterials[selectedFilmIndex].n);

	// Calculate incident beam path
	const incidentEndX = $derived(centerX + Math.tan(angleRad) * (filmTop - beamStartY));
	const incidentEndY = $derived(filmTop);

	// Calculate reflection angle (same as incident for air-film interface)
	const reflectedStartX = $derived(incidentEndX);
	const reflectedStartY = $derived(filmTop);
	const reflectedEndX = $derived(reflectedStartX + Math.tan(angleRad) * beamLength);
	const reflectedEndY = $derived(reflectedStartY - beamLength);

	// Calculate refracted angle in film using Snell's law
	const sinRefracted = $derived((airN * Math.sin(angleRad)) / filmN);
	const refractedAngleRad = $derived(Math.asin(sinRefracted));

	// Calculate refracted beam path through film
	const refractedStartX = $derived(incidentEndX);
	const refractedStartY = $derived(filmTop);
	const filmThickness = $derived(filmBottom - filmTop);
	const refractedEndX = $derived(refractedStartX + Math.tan(refractedAngleRad) * filmThickness);
	const refractedEndY = $derived(filmBottom);

	// Calculate reflection from substrate
	const substrateReflectedStartX = $derived(refractedEndX);
	const substrateReflectedStartY = $derived(filmBottom);
	const substrateReflectedEndX = $derived(
		substrateReflectedStartX + Math.tan(refractedAngleRad) * filmThickness
	);
	const substrateReflectedEndY = $derived(filmTop);

	// Calculate final transmission into air
	const finalTransmissionStartX = $derived(substrateReflectedEndX);
	const finalTransmissionStartY = $derived(filmTop);
	const finalTransmissionEndX = $derived(finalTransmissionStartX + Math.tan(angleRad) * beamLength);
	const finalTransmissionEndY = $derived(finalTransmissionStartY - beamLength);

	// Calculate wavelengths in pixels for each medium
	const airWavelengthPx = $derived(wavelengthNm * nmToPixels);
	const filmWavelengthPx = $derived((wavelengthNm / filmN) * nmToPixels);

	// Calculate coordinates for the perpendicular reference line
	const referenceLineLength = 150; // Length of the reference line in pixels
	// Calculate position relative to first reflection peak
	const reflectionPeakX = $derived(
		reflectedStartX + airWavelengthPx * 0.75 * Math.cos(angleRad - Math.PI / 2)
	);
	const reflectionPeakY = $derived(
		reflectedStartY + airWavelengthPx * 0.75 * Math.sin(angleRad - Math.PI / 2)
	);

	const perpendicularAngle = $derived(angleRad + Math.PI); // Add 90 degrees for perpendicular
	const referenceLineStartX = $derived(
		reflectionPeakX - Math.cos(perpendicularAngle) * referenceLineLength
	);
	const referenceLineStartY = $derived(
		reflectionPeakY - Math.sin(perpendicularAngle) * referenceLineLength
	);
	const referenceLineEndX = $derived(
		reflectionPeakX + Math.cos(perpendicularAngle) * referenceLineLength
	);
	const referenceLineEndY = $derived(
		reflectionPeakY + Math.sin(perpendicularAngle) * referenceLineLength
	);

	// Calculate phase accumulated by refracted beam at substrate interface
	const refractedPathLength = $derived(
		Math.sqrt(
			Math.pow(refractedEndX - refractedStartX, 2) + Math.pow(refractedEndY - refractedStartY, 2)
		)
	);
	const refractedPhaseAtEnd = $derived((refractedPathLength * 2 * Math.PI) / filmWavelengthPx);

	// Phase shift for substrate reflection:
	// 1. Start with the phase from the refracted beam at the intersection
	// 2. Add π if reflecting from higher index material
	// 3. Negate the phase because we're starting from the end point
	const substrateReflectionPhaseShift = $derived(
		-refractedPhaseAtEnd + (filmN < substrateN ? Math.PI : 0)
	);

	// Calculate phase accumulated by substrate reflection at film-air interface
	const substrateReflectionPathLength = $derived(
		Math.sqrt(
			Math.pow(substrateReflectedEndX - substrateReflectedStartX, 2) +
				Math.pow(substrateReflectedEndY - substrateReflectedStartY, 2)
		)
	);
	const substrateReflectionPhaseAtEnd = $derived(
		substrateReflectionPhaseShift - (substrateReflectionPathLength * 2 * Math.PI) / filmWavelengthPx
	);

	// Final transmission should match the phase of substrate reflection at the interface
	const finalTransmissionPhaseShift = $derived(substrateReflectionPhaseAtEnd);
</script>

<svelte:head>
	<title>Thin Film Simulation</title>
	<meta
		name="description"
		content="A simulation of light interacting with a thin film on a substrate"
	/>
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
</svelte:head>

<div class="simulation-container">
	<h1>Thin Film Physics Simulation</h1>

	<div class="controls">
		<div class="control-group">
			<label for="angle-slider">
				Incident Angle: {angleDegrees}°
			</label>
			<input
				id="angle-slider"
				type="range"
				min={angleRange.min}
				max={angleRange.max}
				bind:value={angleDegrees}
				class="slider"
			/>
		</div>

		<div class="control-group">
			<label for="film-select">Film Material:</label>
			<select id="film-select" bind:value={selectedFilmIndex} class="dropdown">
				{#each filmMaterials as material, index}
					<option value={index}>{material.label} (n = {material.n})</option>
				{/each}
			</select>
		</div>

		<div class="control-group">
			<label for="thickness-slider">
				Film Thickness: {filmThicknessNm} nm
			</label>
			<input
				id="thickness-slider"
				type="range"
				min={filmThicknessRange.min}
				max={filmThicknessRange.max}
				bind:value={filmThicknessNm}
				class="slider"
			/>
		</div>
	</div>

	<div class="svg-container">
		<svg width="100%" height="100%" viewBox="0 0 {svgWidth} {svgHeight}" class="simulation-svg">
			<!-- Arrow markers -->
			<defs>
				<marker
					id="arrowhead-red"
					markerWidth="10"
					markerHeight="7"
					refX="9"
					refY="3.5"
					orient="auto"
				>
					<polygon points="0 0, 10 3.5, 0 7" fill="#FF0000" />
				</marker>
				<marker
					id="arrowhead-pink"
					markerWidth="10"
					markerHeight="7"
					refX="9"
					refY="3.5"
					orient="auto"
				>
					<polygon points="0 0, 10 3.5, 0 7" fill="#FF6B6B" />
				</marker>
				<marker
					id="arrowhead-green"
					markerWidth="10"
					markerHeight="7"
					refX="9"
					refY="3.5"
					orient="auto"
				>
					<polygon points="0 0, 10 3.5, 0 7" fill="#00AA00" />
				</marker>
				<marker
					id="arrowhead-purple"
					markerWidth="10"
					markerHeight="7"
					refX="9"
					refY="3.5"
					orient="auto"
				>
					<polygon points="0 0, 10 3.5, 0 7" fill="#6600CC" />
				</marker>
				<marker
					id="arrowhead-blue"
					markerWidth="10"
					markerHeight="7"
					refX="9"
					refY="3.5"
					orient="auto"
				>
					<polygon points="0 0, 10 3.5, 0 7" fill="#0066CC" />
				</marker>
			</defs>

			<!-- Background -->
			<rect width="100%" height="100%" fill="#f8f9fa" />

			<!-- Air region label -->
			<text x="50" y="30" class="region-label">Air (n = {airN})</text>

			<!-- Film layer -->
			<rect
				x="0"
				y={filmTop}
				width="100%"
				height={filmBottom - filmTop}
				fill="#87CEEB"
				stroke="#4682B4"
				stroke-width="2"
			/>
			<text x="50" y={filmTop + 25} class="region-label">
				{filmMaterials[selectedFilmIndex].label} Film (n = {filmN.toFixed(2)})
			</text>

			<!-- Substrate layer -->
			<rect
				x="0"
				y={substrateTop}
				width="100%"
				height={substrateBottom - substrateTop}
				fill="#D3D3D3"
				stroke="#A9A9A9"
				stroke-width="2"
			/>
			<text x="50" y={substrateTop + 25} class="region-label">
				Glass Substrate (n = {substrateN})
			</text>

			<!-- Reference line -->
			<line
				x1={referenceLineStartX}
				y1={referenceLineStartY}
				x2={referenceLineEndX}
				y2={referenceLineEndY}
				stroke="#888888"
				stroke-width="2"
				opacity="0.5"
			/>

			<!-- Light beams -->

			<!-- incident beam -->
			<LightBeam
				x1={centerX}
				y1={beamStartY}
				x2={incidentEndX}
				y2={incidentEndY}
				color="#FF0000"
				arrowId="arrowhead-red"
				label="Incident"
				labelOffset={{ x: -80, y: 20 }}
				{wavelengthNm}
				refractiveIndex={airN}
				{nmToPixels}
				waveStart="end"
			/>

			<!-- reflected beam -->
			<LightBeam
				x1={reflectedStartX}
				y1={reflectedStartY}
				x2={reflectedEndX}
				y2={reflectedEndY}
				color="#FF6B6B"
				arrowId="arrowhead-pink"
				label="Reflected"
				labelOffset={{ x: -60, y: 20 }}
				{wavelengthNm}
				refractiveIndex={airN}
				{nmToPixels}
				waveStart="start"
			/>

			<!-- refracted beam -->
			<LightBeam
				x1={refractedStartX}
				y1={refractedStartY}
				x2={refractedEndX}
				y2={refractedEndY}
				color="#00AA00"
				arrowId="arrowhead-green"
				{wavelengthNm}
				refractiveIndex={filmN}
				{nmToPixels}
				waveStart="start"
			/>

			<!-- substrate reflected beam -->
			<LightBeam
				x1={substrateReflectedStartX}
				y1={substrateReflectedStartY}
				x2={substrateReflectedEndX}
				y2={substrateReflectedEndY}
				color="#6600CC"
				arrowId="arrowhead-purple"
				{wavelengthNm}
				refractiveIndex={filmN}
				{nmToPixels}
				waveStart="end"
				phaseShift={finalTransmissionPhaseShift}
			/>

			<!-- transmitted beam -->
			<LightBeam
				x1={finalTransmissionStartX}
				y1={finalTransmissionStartY}
				x2={finalTransmissionEndX}
				y2={finalTransmissionEndY}
				color="#0066CC"
				arrowId="arrowhead-blue"
				label="Transmitted"
				labelOffset={{ x: 20, y: 10 }}
				{wavelengthNm}
				refractiveIndex={airN}
				{nmToPixels}
				waveStart="start"
				phaseShift={finalTransmissionPhaseShift * -1}
			/>
		</svg>
	</div>

	<div class="info-panel">
		<h3>Physics Information</h3>
		<p><strong>Wavelength:</strong> {wavelengthNm} nm</p>
		<p><strong>Incident Angle:</strong> {angleDegrees}°</p>
		<p>
			<strong>Refracted Angle in Film:</strong>
			{((refractedAngleRad * 180) / Math.PI).toFixed(1)}°
		</p>
		<p><strong>Film Thickness:</strong> {filmThicknessNm} nm</p>
		<p><strong>Film Material:</strong> {filmMaterials[selectedFilmIndex].label} (n = {filmN})</p>
	</div>
</div>

<style>
	.simulation-container {
		max-width: 1000px;
		margin: 0 auto;
		padding: 1rem;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
	}

	h1 {
		text-align: center;
		color: #333;
		margin-bottom: 2rem;
	}

	.controls {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
		gap: 1rem;
		margin-bottom: 2rem;
		background: #f8f9fa;
		padding: 1.5rem;
		border-radius: 8px;
		border: 1px solid #e9ecef;
	}

	.control-group {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	label {
		font-weight: 600;
		color: #495057;
		font-size: 0.9rem;
	}

	.slider {
		width: 100%;
		height: 6px;
		border-radius: 3px;
		background: #ddd;
		outline: none;
		-webkit-appearance: none;
	}

	.slider::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 20px;
		height: 20px;
		border-radius: 50%;
		background: #007bff;
		cursor: pointer;
		border: 2px solid white;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
	}

	.slider::-moz-range-thumb {
		width: 20px;
		height: 20px;
		border-radius: 50%;
		background: #007bff;
		cursor: pointer;
		border: 2px solid white;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
	}

	.dropdown {
		padding: 0.5rem;
		border: 1px solid #ced4da;
		border-radius: 4px;
		background: white;
		font-size: 0.9rem;
		cursor: pointer;
	}

	.dropdown:focus {
		outline: none;
		border-color: #007bff;
		box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
	}

	.svg-container {
		background: white;
		border: 1px solid #e9ecef;
		border-radius: 8px;
		margin-bottom: 2rem;
		overflow: hidden;
	}

	.simulation-svg {
		display: block;
		max-width: 100%;
		height: auto;
	}

	.region-label {
		font-size: 14px;
		font-weight: 600;
		fill: #333;
	}

	.beam-label {
		font-size: 12px;
		font-weight: 500;
		fill: #333;
	}

	.info-panel {
		background: #f8f9fa;
		padding: 1.5rem;
		border-radius: 8px;
		border: 1px solid #e9ecef;
	}

	.info-panel h3 {
		margin-top: 0;
		color: #495057;
	}

	.info-panel p {
		margin: 0.5rem 0;
		color: #6c757d;
	}

	/* Mobile optimizations */
	@media (max-width: 768px) {
		.simulation-container {
			padding: 0.5rem;
		}

		h1 {
			font-size: 1.5rem;
		}

		.controls {
			grid-template-columns: 1fr;
			padding: 1rem;
		}

		.region-label {
			font-size: 12px;
		}

		.beam-label {
			font-size: 10px;
		}

		.info-panel {
			padding: 1rem;
		}
	}

	@media (max-width: 480px) {
		.simulation-container {
			padding: 0.25rem;
		}

		h1 {
			font-size: 1.25rem;
			margin-bottom: 1rem;
		}

		.controls {
			padding: 0.75rem;
		}

		.info-panel {
			padding: 0.75rem;
		}

		.info-panel p {
			font-size: 0.9rem;
		}
	}
</style>
