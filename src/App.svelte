<script>

	let term = "";
	let data = null;
	let showUnavailable = false;
	let loading = false;

	const usp = new URLSearchParams(location.search);
	
	const get = (q) => fetch(( usp.get("api") || "https://libsearch-2imy5ahygq-km.a.run.app" ) + "?query=" + encodeURIComponent(q)).then(r => r.json());

	const doSearch = async () => {

		loading = true;

		const j = await get(term);
		let allResults = [];

		for (const libresult of j.Overdrive) {
			for (const key of Object.keys(libresult.data)) {
				allResults.push({
					__resultId: key,
					__url: `https://link.overdrive.com/?websiteId=${libresult.library}&titleId=${key}`,
					...libresult.data[key]
				});
			}
		}

		data = allResults;
		window.last = allResults;

		loading = false;

	}

	const handleEnter = e => {
		if (e.keyCode === 13) {
			!loading && doSearch();
		}
	}

</script>

<main>

	<div class="container">

		<h1>📚 Search any book</h1>

		<input placeholder="Harry Potter and the Chamber of Secrets" type="text" on:keyup={handleEnter} bind:value={term} />
		<button disabled={loading} on:click={doSearch}>{loading ? "Loading...": "Go"}</button>

		{#if data}
			<h2>{data.length} results ({data.filter(x => x.isAvailable).length} available)</h2>
			<label>
				<input type="checkbox" bind:checked={showUnavailable} /> Show unavailable results
			</label>

			{#each data as item}
				{#if item.isAvailable || showUnavailable}
					
					<div class={"book" + (item.isAvailable ? "":" unavailable")}>

						<div class="left">

							<img src={item.covers.cover150Wide.href} alt={item.title + " cover"} />

							<div class="info">
								<h4>{item.title}</h4>
								<i>{item.firstCreatorName}</i>



								{#if item.formats.map(x => x["id"]).includes("ebook-kindle")}
									<span class="kindle"> (Works with Kindle)</span>
								{/if}

							</div>

						</div>

						

						<div class="action">
							<a target="_blank" rel="noreferrer" class="button" href={item.__url}>Access</a>
						</div>

					</div>

				{/if}
			{/each}
		{/if}

	</div>

</main>

<style>

	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: start;
	}

	.left {
		display: flex;
		flex-direction: row;
		justify-content: flex-start;
		align-items: center;
	}

	.left img {
		height: 50px;
		margin-right: 10px;
	}

	.kindle {
		color: rgb(255, 154, 1);
	}

	.book {
		display: flex;
		align-items: center;
		justify-content: space-between;
		flex-flow: row nowrap;

		border: 1px solid #EEE;
		padding: 1em;
		margin: 1em 0;
		border-radius: 3px;
	}

	.book.unavailable {
		opacity: 0.5;
		border: 1px dashed red;
	}

	h4 {margin: 0; margin-bottom: 5px;}

	a.button {
		display: inline-block;
		padding: 0.5em 1em;
		background: #f0f0f0;
		border: 1px solid #ccc;
		border-radius: 3px;
		text-decoration: none;
		color: #333;
		font-size: 1em;
		font-weight: bold;
	}

	.container {
		width: 500px;
	}

	input[type=text], button {
		display: block;
		width: 500px;
	}

	input[type=text] {
		font-size: 1.5em;
		padding: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
	}

	@media (max-width: 500px) {

		.container {
			width: 100%;
		}

		input[type=text], button {
			width: 100%;
		}
	}

</style>
