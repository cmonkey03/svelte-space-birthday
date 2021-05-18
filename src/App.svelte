<script>
	let date, promise;
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

		const imageUrls = await fetch(`https://epic.gsfc.nasa.gov/api/enhanced/date/${foundDate}`)
			.then(res => res.json())
			.then(data => {
				const pathDate = foundDate.replaceAll('-', '/');
				
				return data.map((item) => {
					return `https://epic.gsfc.nasa.gov/archive/natural/${pathDate}/png/${item.image}`
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
</script>

<h1 style="text-align: center">Hello Sveltian Space Voyager!</h1>
<p>Please enter your date-of-birth, and we'll ask NASA to show us earth on your most recent birthday!</p> 
<input
	type="date"
	min="1899-01-01"
	max={today}
	bind:value={date}
	on:change={event => handleChange(event)} 
	>

{#await promise}
	<!-- optionally show something while promise is pending -->
	<h2>Asking NASA for your birthday picture...</h2>
{:then links}
	{#each links as link, i}
		<img
			alt="Your bEARTHday ${i}"
			style="height: 50%; width: 50%;"
			src={link}
		/>
	{/each}
{:catch error}
	<!-- optionally show something while promise was rejected -->
	<p style="color: red">Oops! There seems to be an issue, please try again.</p>
{/await}