<script>
	// @ts-nocheck
	import { onMount } from 'svelte';

	let editorElement;
	let uploadElement;
	let editorJs;
	let aceEditor;
	let isProgrammaticUpdate = false;
	let isError = false;
	let loading = true;

	onMount(async () => {
		await loadLibraries();

		setTimeout(() => {
			loading = false;
		}, 1000);

		aceEditor = ace.edit('jsonOutput');
		aceEditor.getSession().on('change', async () => {
			if (isProgrammaticUpdate) return;
			const newContent = aceEditor.getValue();

			try {
				isProgrammaticUpdate = true;
				const jsonData = JSON.parse(newContent);
				await editorJs.isReady;
				editorJs
					.render(jsonData)
					.then(() => {
						isError = false;
					})
					.catch((err) => {
						isError = true;
					})
					.finally(() => {
						isProgrammaticUpdate = false;
					});
			} catch (error) {
				isError = true;
				isProgrammaticUpdate = false;
			}
		});

		editorJs = new EditorJS({
			holder: 'editorjs',
			tools: {
				header: { class: Header, inlineToolbar: true },
				paragraph: { class: Paragraph, inlineToolbar: true },
				list: { class: EditorjsList, inlineToolbar: true },
				delimiter: { class: Delimiter, inlineToolbar: true },
				quote: { class: Quote, inlineToolbar: true },
				code: { class: CodeTool, inlineToolbar: true },
				table: { class: Table, inlineToolbar: true },
				embed: { class: Embed, inlineToolbar: true },
				underline: { class: Underline, inlineToolbar: true },
				marker: { class: Marker, inlineToolbar: true },
				checklist: { class: Checklist, inlineToolbar: true },
				link: { class: LinkTool, inlineToolbar: true },
				image: SimpleImage
			},
			onChange: async () => {
				if (isProgrammaticUpdate) return;
				const output = await editorJs.save();
				isProgrammaticUpdate = true;
				aceEditor.setValue(JSON.stringify(output, null, 2), -1);
				isError = false;
				isProgrammaticUpdate = false;
			}
		});
	});

	const loadScript = (src) => {
		return new Promise((resolve, reject) => {
			const script = document.createElement('script');
			script.src = src;
			script.defer = true;
			script.onload = resolve;
			script.onerror = reject;
			document.head.appendChild(script);
		});
	};

	const loadLibraries = async () => {
		try {
			await Promise.all([
				loadScript('https://cdnjs.cloudflare.com/ajax/libs/ace/1.39.1/ace.min.js'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/editorjs@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/header@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/list@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/paragraph@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/image@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/delimiter@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/quote@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/code@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/table@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/embed@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/underline@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/marker@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/checklist@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/link@latest'),
				loadScript('https://cdn.jsdelivr.net/npm/@editorjs/simple-image@latest')
			]);
			console.log('All EditorJS tools loaded successfully!');
		} catch (error) {
			console.error('Error loading libraries: ', error);
		}
	};

	const downloadFile = async () => {
		if (editorJs) {
			const output = await editorJs.save();
			const jsonData = JSON.stringify(output, null, 2);
			const blob = new Blob([jsonData], { type: 'application/json' });
			const url = URL.createObjectURL(blob);

			const a = document.createElement('a');
			a.href = url;
			a.download = 'editorjs-output.json';
			document.body.appendChild(a);
			a.click();
			document.body.removeChild(a);

			URL.revokeObjectURL(url);
		}
	};

	const importJson = (event) => {
		const file = event.target.files[0];
		if (!file) return;

		const reader = new FileReader();
		reader.onload = async (e) => {
			try {
				const jsonData = JSON.parse(e.target.result);
				aceEditor.setValue(JSON.stringify(jsonData, null, 2), -1);
			} catch (error) {
				console.error('Invalid JSON file:', error);
				alert('Invalid JSON file. Please check the format.');
			}
		};
		reader.readAsText(file);
	};
</script>

<div class="vh-100" style="display: flex; flex-direction: column; overflow: auto">
	<div style="background: #2c3e50" class="p-3 d-flex justify-content-between align-items-center">
		<div class="text-white">EditorJS</div>

		<div class="d-flex align-items-center" style="gap: 8px">
			<button class="btn btn-primary" on:click={() => uploadElement.click()}>+ Import JSON</button>

			<button class="btn btn-success" on:click={downloadFile}>💾 Save as JSON</button>
		</div>
	</div>

	<input
		bind:this={uploadElement}
		style="display: none;"
		type="file"
		accept="application/json"
		on:change={importJson}
	/>

	<div class="row grow" style="margin-left: unset; margin-right: unset">
		<!-- Sidebar (Editor) -->
		<div class="col-md-6 p-4">
			<h5>✏️ Editor</h5>
			<div class="border rounded bg-white text-dark p-3" style="height: 84vh; overflow-y: auto; padding-left: 44px !important">
				<div bind:this={editorElement} id="editorjs"></div>
			</div>
		</div>

		<!-- Output (JSON Output) -->
		<div class="col-md-6 p-4">
			<h5>📦 JSON Output</h5>
			<div
				class="border rounded bg-white text-dark {isError ? 'border-danger' : ''}"
				style="height: 84vh; overflow-y: auto;"
			>
				<div id="jsonOutput" class="h-100"></div>
			</div>
		</div>
	</div>
</div>

{#if loading}
	<div class="loading-screen">
		<div class="loading-text">Loading...</div>
	</div>
{/if}

<style>
	.loading-screen {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(255, 255, 255, 0.7); /* White background with transparency */
		backdrop-filter: blur(5px); /* Blur effect */
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 9999;
	}

	/* Loading text styling */
	.loading-text {
		font-size: 1.25rem;
		font-weight: bold;
		color: #000;
		animation: bounceText 1s ease-in-out infinite;
	}

	@keyframes bounceText {
		0%,
		100% {
			transform: translateY(0); /* Start and end at normal position */
		}
		50% {
			transform: translateY(-10px); /* Move up by 20px at the middle of the animation */
		}
	}

	@keyframes fadeOut {
		0% {
			opacity: 1; /* Fully visible */
		}
		100% {
			opacity: 0; /* Fully invisible */
			display: none;
		}
	}
</style>
