<script>
	import Cell from './Cell.svelte';
	import Image from './Image.svelte';

	// 1. user(base)
	let user = null;
	const { userbase } = window; window.userbase = null;

	// 2. db stuff
	const databaseName = 'files';
	let fileItems = [];
	const changeHandler = (items) => fileItems = items;

	// 3. init
	let authPromise = userbase.init({ appId: __myapp.env.BASE_APP_ID })
		.then(async (session) => {
			user = session.user;
			if (!user) await userbase.signUp({ username: 'yummyy', password: 'yummyy' }).then((u) => user = u);
			await userbase.openDatabase({ databaseName, changeHandler });
		});

	// 4. handle upload
	let uploadFilePromise;
	function uploadFile(e) {
		const file = e.target.files[0];
		const randomId = Math.random().toString().substring(2);
		uploadFilePromise = userbase.insertItem({ databaseName, item: randomId })
			.then(() => {
				const {itemId} = fileItems.filter(({fileId, item}) => randomId === item && !fileId)[0];
				userbase.uploadFile({ databaseName, itemId, file });
			});
	}

	// 5. get file
	let filesCache = {};
	async function getFile(fileId) {
		const { file } = await userbase.getFile({ databaseName, fileId });
		filesCache[fileId] = file;
		return file;
	}

	// 6. pagination
	let atBottom = false;
	let imagesToShow = 2;
	// $: console.log(atBottom, imagesToShow);
	$: if(atBottom && imagesToShow < fileItems.length) {
		imagesToShow += 1;
	}
</script>

<svelte:window on:scroll={() => atBottom = window.innerHeight + window.pageYOffset >= document.documentElement.scrollHeight} />

<Cell promise={authPromise}>
	Hi, {user.username}!
	<Cell promise={uploadFilePromise}>
		<input type="file" on:change={uploadFile}>
	</Cell>
	{#each fileItems.slice(0, imagesToShow) as {fileId, fileName}}
		{#if filesCache[fileId]}
			<Image file={filesCache[fileId]} alt={fileName} />
		{:else}
			<Cell promise={getFile(fileId)} let:response>
				<Image file={response} alt={fileName} />
			</Cell>
		{/if}
	{/each}
</Cell>
