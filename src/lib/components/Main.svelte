<script>
	// @ts-nocheck
	import { onMount } from 'svelte';
	import continentData from 'country-json/src/country-by-continent.json';
	import capitalData from 'country-json/src/country-by-capital-city.json';
	import languageData from 'country-json/src/country-by-languages.json';

	let combinedDataByContinent = {};
	let filteredData = [];
	let searchQuery = '';

	onMount(listData);

	function listData() {
		combinedDataByContinent = languageData.reduce((acc, languageItem) => {
			const matchingContinentItem = continentData.find(
				(continentItem) => continentItem.country === languageItem.country
			);
			const matchingCityItem = capitalData.find(
				(cityItem) => cityItem.country === languageItem.country
			);
			const combinedItem = {
				...languageItem,
				...(matchingContinentItem || {}),
				...(matchingCityItem || {})
			};

			if (combinedItem.continent !== undefined && (combinedItem.country || combinedItem.city)) {
				acc[combinedItem.continent] = acc[combinedItem.continent] || [];
				acc[combinedItem.continent].push(combinedItem);
			}
			return acc; // Return the updated accumulator
		}, {});

		// Apply search when the data is updated
		search();
	}

	function search() {
		const lowerCaseQuery = searchQuery.toLowerCase();

		filteredData = Object.entries(combinedDataByContinent).reduce(
			(acc, [continent, continentData]) => {
				const filteredContinentData = continentData.filter((item) => {
					return (
						(item.country && item.country.toLowerCase().includes(lowerCaseQuery)) ||
						(item.city && item.city.toLowerCase().includes(lowerCaseQuery))
					);
				});

				if (filteredContinentData.length > 0) {
					acc.push({
						continent,
						data: filteredContinentData
					});
				}

				return acc;
			},
			[]
		);
	}
</script>

<svelte:head>
	<title>Search by country name or capital name</title>
</svelte:head>

<main class="container">
	<div class="row">
		<div class="col-7 layout__right">
			<h1>Browse continents</h1>
			<p>To view country and see capital city and languages spoken.</p>
			{#each filteredData as { continent, data }}
				<details class="level-one">
					<summary>
						{continent}
					</summary>
					<h2 class="countries">Countries</h2>
					{#each data as item (item)}
						<details class="level-two">
							<summary>{item.country}</summary>
							<p class="level-two__inner">Language spoken: {item.languages.join(', ')}</p>
							<p class="level-two__inner">Capital city: {item.city}</p>
						</details>
					{/each}
				</details>
			{:else}
				<p>No results found.</p>
			{/each}
		</div>
		<aside class="col-5 layout__left">
			<h2>Search</h2>
			<p>by country name or capital name</p>

			<input
				bind:value={searchQuery}
				placeholder="Search"
				on:input={search}
				class="input__search"
			/>
		</aside>
	</div>
	<div class="semi-circle"></div>
</main>

<style lang="scss">
	@import 'simple-grid.min.css';

	$derby: #ffefd8;
	$solitaire: #fef5e8;
	$darkPurple: #1c1150;
	$lightGrey: lightgrey;

	h1,
	h2,
	h3,
	p {
		margin: 0.2rem;
		color: $darkPurple;
		line-height: 2rem;
		font-weight: 400;
	}

	// override grid
	.row .layout__right,
	.row .layout__left {
		margin-top: 0;
		padding-top: 1rem;
	}

	.layout__left {
		p {
			margin: 0;
		}
	}

	.layout__right {
		padding-top: 1rem;
		p {
			margin-bottom: 1rem;
		}
	}

	aside {
		padding: 1rem;
		background-color: $solitaire;
		height: 100vh;
	}

	.countries {
		margin: 1rem 0 1rem 20px;
		font-size: 1.2rem;
	}

	summary {
		font-size: 1.5rem;
		&::marker {
			color: $darkPurple;
		}
		border: 1px solid $lightGrey;
		border-radius: 0.2rem;
		background-color: $solitaire;
		padding: 0.5rem;
		margin-bottom: 0.2rem;
		h3 {
			margin: 0;
		}
	}

	details {
		padding: 0.5rem;
		margin-bottom: 0.5rem;
	}

	.level-one {
		background-color: $derby;
	}

	.level-two {
		background-color: lighten($derby, 10%);
		margin-left: 20px;

		&__inner {
			margin: 0.5rem 0 0 0.5rem;
		}

		summary {
			background-color: lighten($derby, 10%);
			font-size: 1.2rem;
			margin: 0;
			padding: 0.5rem;
		}
	}

	.input__search {
		width: 100%;
		margin: 1rem 0 0.2rem 0;
		padding: 0.5rem;
		border-radius: 0.2rem;
		font-size: 1rem;
	}
</style>
