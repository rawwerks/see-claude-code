<script lang="ts">
	import { onMount } from 'svelte';
	import Button from '$lib/components/ui/button/button.svelte';
	import Card from '$lib/components/ui/card/card.svelte';
	import * as CardContent from '$lib/components/ui/card/index.js';
	import Textarea from '$lib/components/ui/textarea/textarea.svelte';
	import { ChevronLeft, ChevronRight, ThumbsUp, ThumbsDown, Upload, Download } from 'lucide-svelte';

	let traces = $state([]);
	let currentTraceIndex = $state(0);
	let feedback = $state({});
	let dragover = $state(false);

	let currentTrace = $derived(traces[currentTraceIndex]);
	let currentTraceId = $derived(currentTrace?.filename || currentTraceIndex);
	let currentFeedback = $derived(feedback[currentTraceId] || { vote: null, notes: '' });

	function handleFiles(files: FileList) {
		Array.from(files).forEach(file => {
			if (file.name.endsWith('.jsonl')) {
				const reader = new FileReader();
				reader.onload = (e) => {
					const content = e.target?.result as string;
					const spans = content.trim().split('\n').map(line => {
						try {
							return JSON.parse(line);
						} catch {
							return null;
						}
					}).filter(Boolean);
					
					// Each .jsonl file is one trace containing multiple spans
					const trace = {
						filename: file.name,
						spans: spans
					};
					traces = [...traces, trace];
				};
				reader.readAsText(file);
			}
		});
	}

	function handleDrop(e: DragEvent) {
		e.preventDefault();
		dragover = false;
		if (e.dataTransfer?.files) {
			handleFiles(e.dataTransfer.files);
		}
	}

	function handleDragOver(e: DragEvent) {
		e.preventDefault();
		dragover = true;
	}

	function handleDragLeave(e: DragEvent) {
		e.preventDefault();
		dragover = false;
	}

	function handleFileInput(e: Event) {
		const files = (e.target as HTMLInputElement).files;
		if (files) {
			handleFiles(files);
		}
	}

	function navigateTrace(direction: number) {
		const newIndex = currentTraceIndex + direction;
		if (newIndex >= 0 && newIndex < traces.length) {
			currentTraceIndex = newIndex;
		}
	}

	function vote(value: number) {
		const traceId = currentTraceId;
		if (!traceId) return;
		if (!feedback[traceId]) {
			feedback[traceId] = { vote: null, notes: '' };
		}
		feedback[traceId].vote = feedback[traceId].vote === value ? null : value;
	}

	function updateNotes(notes: string) {
		const traceId = currentTraceId;
		if (!traceId) return;
		if (!feedback[traceId]) {
			feedback[traceId] = { vote: null, notes: '' };
		}
		feedback[traceId].notes = notes;
	}

	function exportFeedback() {
		const exportData = Object.entries(feedback).map(([traceId, data]) => ({
			traceId,
			...data
		}));
		const blob = new Blob([JSON.stringify(exportData, null, 2)], { type: 'application/json' });
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = 'feedback.json';
		a.click();
		URL.revokeObjectURL(url);
	}

	function getMessageTypeColor(type: string, role?: string) {
		if (type === 'user') return 'bg-blue-50 border-blue-200';
		if (type === 'assistant') return 'bg-green-50 border-green-200';
		if (role === 'assistant' && type !== 'user') return 'bg-green-50 border-green-200';
		return 'bg-gray-50 border-gray-200';
	}

	onMount(() => {
		function handleKeydown(e: KeyboardEvent) {
			switch (e.key) {
				case 'ArrowLeft':
					e.preventDefault();
					navigateTrace(-1);
					break;
				case 'ArrowRight':
					e.preventDefault();
					navigateTrace(1);
					break;
				case 'ArrowUp':
					e.preventDefault();
					vote(1);
					break;
				case 'ArrowDown':
					e.preventDefault();
					vote(0);
					break;
			}
		}
		window.addEventListener('keydown', handleKeydown);
		return () => window.removeEventListener('keydown', handleKeydown);
	});
</script>

<div class="min-h-screen bg-gray-100 p-4">
	<div class="max-w-6xl mx-auto">
		<h1 class="text-3xl font-bold text-center mb-8">Claude Code Log Annotator</h1>
		
		{#if traces.length === 0}
			<Card class="p-8">
				<CardContent.Root>
					<div 
						class="border-2 border-dashed border-gray-300 rounded-lg p-12 text-center transition-colors {dragover ? 'border-blue-500 bg-blue-50' : ''}"
						role="button"
						tabindex="0"
						ondrop={handleDrop}
						ondragover={handleDragOver}
						ondragleave={handleDragLeave}
					>
						<Upload class="mx-auto mb-4 h-12 w-12 text-gray-400" />
						<p class="text-xl text-gray-600 mb-4">Drag and drop .jsonl files here</p>
						<p class="text-gray-500 mb-6">or</p>
						<label for="file-input" class="cursor-pointer">
							<Button>Choose Files</Button>
						</label>
						<input 
							id="file-input" 
							type="file" 
							multiple 
							accept=".jsonl"
							class="hidden"
							onchange={handleFileInput}
						/>
					</div>
				</CardContent.Root>
			</Card>
		{:else}
			<!-- Navigation -->
			<div class="flex gap-4 mb-6">
				<Button variant="outline" onclick={() => navigateTrace(-1)} disabled={currentTraceIndex === 0}>
					<ChevronLeft class="h-4 w-4 mr-2" />
					Previous
				</Button>
				<div class="flex-1 text-center py-2">
					<span class="text-lg font-medium">
						Trace {currentTraceIndex + 1} of {traces.length}
					</span>
				</div>
				<Button variant="outline" onclick={() => navigateTrace(1)} disabled={currentTraceIndex === traces.length - 1}>
					Next
					<ChevronRight class="h-4 w-4 ml-2" />
				</Button>
			</div>

			<!-- Centered Title -->
			<h2 class="text-2xl font-bold text-center mb-8">
				Conversation Trace: {currentTrace?.filename || 'Unknown'}
			</h2>

			<!-- Full Width Conversation Display -->
			<Card class="mb-8">
				<CardContent.Root class="p-6">
					{#if currentTrace?.spans}
						<div class="space-y-4 max-h-96 overflow-y-auto">
							{#each currentTrace.spans as span, index}
								<div class="p-4 rounded-lg border {getMessageTypeColor(span.type, span.message?.role)}">
									<div class="flex justify-between items-start mb-2">
										<span class="text-sm font-medium text-gray-600 flex-shrink-0">
											Span {index + 1}: {span.type} {span.message?.role ? `(${span.message.role})` : ''}
										</span>
										<span class="text-xs text-gray-500 flex-shrink-0 ml-2">
											{span.timestamp ? new Date(span.timestamp).toLocaleString() : 'No timestamp'}
										</span>
									</div>
									{#if span.message?.content}
										{#if Array.isArray(span.message.content)}
											{#each span.message.content as content}
												{#if content.type === 'text'}
													<p class="text-gray-800 break-words whitespace-pre-wrap overflow-wrap-anywhere">{content.text}</p>
												{:else if content.type === 'tool_use'}
													<div class="bg-gray-100 p-3 rounded mt-2 overflow-hidden">
														<p class="text-sm font-medium break-words">Tool: {content.name}</p>
														<pre class="text-xs mt-1 overflow-x-auto whitespace-pre-wrap break-all">{JSON.stringify(content.input, null, 2)}</pre>
													</div>
												{/if}
											{/each}
										{:else}
											<p class="text-gray-800 break-words whitespace-pre-wrap overflow-wrap-anywhere">{span.message.content}</p>
										{/if}
									{:else if span.summary}
										<p class="text-gray-800 font-medium break-words">{span.summary}</p>
									{:else if span.error}
										<p class="text-red-600 font-medium break-words">Error: {span.error}</p>
									{:else}
										<p class="text-gray-500 italic">No content available</p>
									{/if}
								</div>
							{/each}
						</div>
					{/if}
				</CardContent.Root>
			</Card>

			<!-- Bottom Controls Section -->
			<div class="grid grid-cols-1 md:grid-cols-3 gap-6">
				<!-- Voting -->
				<Card>
					<CardContent.Root class="p-6">
						<h3 class="text-lg font-semibold mb-4 text-center">Rate this trace</h3>
						<div class="flex gap-2 justify-center">
							<Button 
								variant={currentFeedback.vote === 1 ? "default" : "outline"}
								size="sm"
								onclick={() => vote(1)}
							>
								<ThumbsUp class="h-4 w-4 mr-1" />
								Up
							</Button>
							<Button 
								variant={currentFeedback.vote === 0 ? "destructive" : "outline"}
								size="sm"
								onclick={() => vote(0)}
							>
								<ThumbsDown class="h-4 w-4 mr-1" />
								Down
							</Button>
						</div>
					</CardContent.Root>
				</Card>

				<!-- Qualitative Feedback -->
				<Card>
					<CardContent.Root class="p-6">
						<h3 class="text-lg font-semibold mb-4 text-center">Feedback Notes</h3>
						<div>
							<Textarea 
								id="notes-textarea"
								placeholder="Add your qualitative feedback here..."
								value={currentFeedback.notes}
								oninput={(e) => updateNotes(e.target.value)}
								rows={3}
							/>
						</div>
					</CardContent.Root>
				</Card>

				<!-- Export -->
				<Card>
					<CardContent.Root class="p-6">
						<h3 class="text-lg font-semibold mb-4 text-center">Export</h3>
						<Button onclick={exportFeedback} class="w-full">
							<Download class="h-4 w-4 mr-2" />
							Export Feedback
						</Button>
						<p class="text-xs text-gray-500 mt-2 text-center">
							Exports feedback as JSON
						</p>
					</CardContent.Root>
				</Card>
			</div>
		{/if}
	</div>
</div>
