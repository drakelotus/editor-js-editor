<script>
	// @ts-nocheck
	import { onMount } from 'svelte';

	let editorElement;
	let editorJs;
	let aceEditor;

	onMount(() => {
		aceEditor = ace.edit('jsonOuput');
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
				image: { class: ImageTool, inlineToolbar: true }
			},
			onChange: async () => {
				const output = await editorJs.save();
				aceEditor.setValue(JSON.stringify(output, null, 2), -1); // Update Ace Editor
				aceEditor.setReadOnly(true);
			}
		});
	});

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

			URL.revokeObjectURL(url); // Cleanup
		}
	};
</script>

<div class="vh-100" style="display: flex; flex-direction: column; overflow: auto">
	<div style="background: #2c3e50" class="p-3 d-flex justify-content-between align-items-center">
		<div class="text-white">EditorJS Playground</div>
		<button class="btn btn-success" on:click={downloadFile}>💾 Save as JSON</button>
	</div>

	<div class="row grow" style="margin-left: unset; margin-right: unset">
		<!-- Sidebar (Editor) -->
		<div class="col-md-6 p-4">
			<h5>✏️ Editor</h5>
			<div class="border rounded bg-white text-dark p-3" style="height: 84vh; overflow-y: auto;">
				<div bind:this={editorElement} id="editorjs"></div>
			</div>
		</div>

		<!-- Output (JSON Output) -->
		<div class="col-md-6 p-4">
			<h5>📦 JSON Output</h5>
			<div
				class="border rounded bg-white text-dark p-3"
				style="height: 84vh; overflow-y: auto;"
				id="jsonOuput"
			></div>
		</div>
	</div>
</div>

<style>
</style>
