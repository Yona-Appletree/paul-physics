<script lang="ts">
	//
	// Physics simulation of a single ray of light hitting a thin film on a
	// substrate, used to demonstrate the constructive and destructive interference
	// for introductory-level physics classes
	//

	// Non-adjustable parameters
	const wavelengthNm = 500;
	const airN = 1;
	const substrateN = 1.5; // index of refraction

	// User adjustable parameters
	const filmThicknessNm = {
		min: 10,
		max: 700,
		current: 100
	};

	const film = {
		choices: [
			{
				label: 'MgF2',
				n: 1.38
			},
			{
				label: 'SiO2',
				n: 1.65
			}
		],
		currentIndex: 0
	};

	const angleDegrees = {
		min: 0,
		max: 30,
		current: 15 // angle from vertical
	};
</script>

<svelte:head>
	<title>Thin film simulation</title>
	<meta
		name="description"
		content="A simulation of light interacting with a thin film on a substrate"
	/>
</svelte:head>
