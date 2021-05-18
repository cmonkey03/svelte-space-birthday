<script>
	let date, promise;
	let currentCarouselIndex = 0;
	const today = new Date().toISOString().split("T")[0];

	async function getImageUrls() {
		const foundDate = await fetch(`https://epic.gsfc.nasa.gov/api/natural/all`)
			.then(res => res.json())
			.then(data => {
				const dateObj = data.find((item) => {
					if (parseInt(date.slice(5,7)) === parseInt(item.date.slice(5,7))) {
						if (parseInt(date.slice(8,10)) >= parseInt(item.date.slice(8,10))) {
							return true;                                         
						}
					}

					return false;
				});

				return dateObj.date;
			})
			.catch(error => {
				throw new Error(error);
			});

		const imageUrls = await fetch(`https://epic.gsfc.nasa.gov/api/natural/date/${foundDate}`)
			.then(res => res.json())
			.then(data => {
				const pathDate = foundDate.replaceAll('-', '/');
				
				return data.map((item) => {
					return `https://epic.gsfc.nasa.gov/archive/natural/${pathDate}/png/${item.image}.png`
				});
			})
			.catch(error => {
				throw new Error(error);
			});
		
		return imageUrls;
	}

	const handleChange = () => {
		promise = getImageUrls();
	}

	const next = (carouselLength) =>
	  currentCarouselIndex = (currentCarouselIndex + 1) % carouselLength

</script>

<h1 style="text-align: center">Hello Sveltian Space Voyager!</h1>
<p>Please enter your date of birth, and we'll ask NASA to show you Earth on your most recent birthday!</p> 
<input
	type="date"
	min="1899-01-01"
	max={today}
	bind:value={date}
	on:change={event => handleChange(event)} 
	>

{#await promise}
	<h2>Asking NASA for your birthday picture(s)...</h2>
{:then links}
	{#if links}
		<div style="text-align: center;">
			<div >
				<p><b>{links.length} bEARTHday pictures found</b></p>
				{#each links as link, index}
					{#if index == currentCarouselIndex}
						<img
							alt="Your bEARTHday ${index}"
							style="height: 50%; width: 50%;"
							src={link}
						/>
					{/if}
				{/each}
			</div>
			<button on:click={next(links.length)}>Next</button>
		</div>
	{/if}
{:catch error}
	<p style="color: red">Oops! There seems to be an issue, please try again.</p>
{/await}