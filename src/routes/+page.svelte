<script lang="ts">
	import * as Card from '$lib/components/ui/card/index.js';
	import * as Dialog from '$lib/components/ui/dialog/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import { Separator } from '$lib/components/ui/separator/index.js';
	import { Label } from '$lib/components/ui/label';
	import { Input } from '$lib/components/ui/input';
	import { Trash2 } from 'lucide-svelte';
	import { onMount } from 'svelte';

	interface Drink {
		alcoholics: string[];
		others: string[];
	}

	let alcoholics: string[] = $state([]);
	let alcoholDialogOpen: boolean = $state(false);
	let alcoholInput: string = $state('');
	let others: string[] = $state([]);
	let otherDialogOpen: boolean = $state(false);
	let otherInput: string = $state('');

	let drink: Drink | undefined = $state();
	let loading: boolean = $state(false);

	onMount(async () => {
		const alcoholicsResult = localStorage.getItem('alcoholics');
		if (alcoholicsResult) {
			alcoholics = JSON.parse(alcoholicsResult);
		}

		const othersResult = localStorage.getItem('others');
		if (othersResult) {
			others = JSON.parse(othersResult);
		}
	});

	async function requestPort() {
		const port = await navigator.serial.requestPort();
		console.log(port);

		await port.open({ baudRate: 9600 });
		console.log(port.readable);

		while (port.readable) {
			const reader = port.readable.getReader();

			try {
				while (true) {
					const { value, done } = await reader.read();
					if (done) {
						// Allow the serial port to be closed later.
						console.log('closed');
						reader.releaseLock();
						break;
					}
					if (value) {
						console.log('Requesting drink...');
						requestDrink();
					}
				}
			} catch (error) {
				// TODO: Handle non-fatal read error.
			}
		}
	}

	function addAlcohol() {
		if (alcoholics.includes(alcoholInput)) {
			return;
		}
		alcoholics.push(alcoholInput);
		alcoholInput = '';
		alcoholDialogOpen = false;
		localStorage.setItem('alcoholics', JSON.stringify(alcoholics));
	}

	function removeAlcohol(alcohol: string) {
		alcoholics = alcoholics.filter((it) => it !== alcohol);
		localStorage.setItem('alcoholics', JSON.stringify(alcoholics));
	}

	function addOther() {
		if (others.includes(otherInput)) {
			return;
		}
		others.push(otherInput);
		otherInput = '';
		otherDialogOpen = false;
		localStorage.setItem('others', JSON.stringify(others));
	}

	function removeOther(other: string) {
		console.log(other);
		others = others.filter((it) => it !== other);
		localStorage.setItem('others', JSON.stringify(others));
	}

	async function requestDrink() {
		if (loading || alcoholics.length == 0 || others.length == 0) {
			return;
		}

		loading = true;
		drink = undefined;
		const data = {
			model: 'qwen2.5:1.5b',
			prompt: buildPrompt(),
			stream: false,
			format: {
				type: 'object',
				properties: {
					alcoholics: {
						type: 'array'
					},
					others: {
						type: 'array'
					}
				},
				required: ['alcoholics', 'others']
			}
		};

		const result = await fetch('http://localhost:11434/api/generate', {
			method: 'POST',
			body: JSON.stringify(data)
		});

		const json = await result.json();
		console.log(json.response);

		drink = JSON.parse(json.response);

		if (drink && drink?.alcoholics.length > 2) {
			drink.alcoholics = drink.alcoholics.slice(0, 2);
		}
		if (drink && drink?.others.length > 3) {
			drink.others = drink.others.slice(0, 3);
		}
		loading = false;
	}

	function buildPrompt(): string {
		let prompt: string =
			'From the two lists I will provide, one with some alcoholics/spirits and one with other types of drinks, You have to say which ones to put in a drink. You can choose a MINIMUM of 1 alcoholic/spirit, and a MAXIMUM of 2. Of the other types of drinks, you can choose a MINIMUM of 1 and a MAXIMUM of 3. It is very important that you follow the minimum and maximum.';
		prompt = prompt.concat(
			`Here is the list of alcoholics/spirits: ${JSON.stringify(alcoholics)}.`
		);
		prompt = prompt.concat(
			`And here is the list of other types of drinks: ${JSON.stringify(others)}.`
		);
		prompt = prompt.concat('Respond using JSON.');
		console.log(prompt);
		return prompt;
	}
</script>

<div class="grid min-h-screen w-screen grid-cols-2 grid-rows-6 bg-purple-900">
	<div class="col-span-2 row-span-2 place-content-center justify-items-center">
		<Card.Root class="min-w-xl  w-1/2">
			<Card.Header>
				<Card.Title>SlatGPT</Card.Title>
				<Card.Description>Angiv lager forneden, tryk på knappen og lad magien ske</Card.Description>
			</Card.Header>
			<Card.Content>
				<Button
					onclick={() => {
						requestPort();
					}}>Request port</Button
				>
				{#if drink}
					<Card.Title>Alkohol</Card.Title>
					<ul class="list-disc">
						{#each drink.alcoholics as alcohol}
							<li class="mx-6 mt-2">{alcohol}</li>
						{/each}
					</ul>
					<Card.Title class="mt-6">Opblanding</Card.Title>
					<ul class="list-disc">
						{#each drink.others as other}
							<li class="mx-6 mt-2">{other}</li>
						{/each}
					</ul>
				{:else}
					<p>Tryk på knappen herunder og lad magien ske...</p>
				{/if}
			</Card.Content>
			<Card.Footer>
				<Button
					class="w-full"
					disabled={loading}
					onclick={() => {
						requestDrink();
					}}>Bestil drink</Button
				>
			</Card.Footer>
		</Card.Root>
	</div>
	<div class="row-span-4 justify-items-center p-16">
		<Card.Root class="w-full">
			<Card.Header>
				<Card.Title>Alkohol</Card.Title>
				<Card.Description>Angiv hvilke slags alkohol, der er på lager</Card.Description>
			</Card.Header>
			<Card.Content>
				<Button
					class="w-full"
					onclick={() => {
						alcoholDialogOpen = true;
					}}>Tilføj Alkohol</Button
				>
				<Separator class="my-4" />
				{#if alcoholics.length > 0}
					{#each alcoholics as alcohol}
						<Card.Root class="mt-6 flex w-full justify-between p-6 first:mt-0">
							<Card.Title class="mt-2">{alcohol}</Card.Title>

							<Button onclick={(_) => removeAlcohol(alcohol)} variant="outline" size="icon">
								<Trash2 />
								<span class="sr-only">Remove {alcohol}</span>
							</Button>
						</Card.Root>
					{/each}
				{:else}
					<p>Tilføj noget alkohol for at begynde...</p>
				{/if}
			</Card.Content>
		</Card.Root>
	</div>
	<div class="row-span-4 justify-items-center p-16">
		<Card.Root class="w-full">
			<Card.Header>
				<Card.Title>Opblanding</Card.Title>
				<Card.Description>Angiv hvilke slags opblandinger, der er på lager</Card.Description>
			</Card.Header>
			<Card.Content>
				<Button
					class="w-full"
					onclick={() => {
						otherDialogOpen = true;
					}}>Tilføj Opblanding</Button
				>
				<Separator class="my-4" />
				{#if others.length > 0}
					{#each others as other}
						<Card.Root class="mt-6 flex w-full justify-between p-6 first:mt-0">
							<Card.Title class="mt-2">{other}</Card.Title>

							<Button onclick={(_) => removeOther(other)} variant="outline" size="icon">
								<Trash2 />
								<span class="sr-only">Remove {other}</span>
							</Button>
						</Card.Root>
					{/each}
				{:else}
					<p>Tilføj en opblanding for at begynde...</p>
				{/if}
			</Card.Content>
		</Card.Root>
	</div>
</div>

<Dialog.Root bind:open={alcoholDialogOpen}>
	<Dialog.Content>
		<Dialog.Header>
			<Dialog.Title>Tilføj alkohol</Dialog.Title>
			<Dialog.Description>Skriv navnet på det alkohol du vil tilføje</Dialog.Description>
			<div class="grid gap-4 py-4">
				<div class="grid grid-cols-4 items-center gap-4">
					<Label for="alcohol" class="text-right">Alkohol</Label>
					<Input
						id="alcohol"
						placeholder="Noget lækkert"
						class="col-span-3"
						bind:value={alcoholInput}
						onkeypress={(event: any) => {
							if (event.key == 'Enter') {
								event.preventDefault();
								addAlcohol();
							}
						}}
					/>
				</div>
			</div>
			<Dialog.Footer>
				<Button onclick={addAlcohol}>Tilføj</Button>
			</Dialog.Footer>
		</Dialog.Header>
	</Dialog.Content>
</Dialog.Root>

<Dialog.Root bind:open={otherDialogOpen}>
	<Dialog.Content>
		<Dialog.Header>
			<Dialog.Title>Tilføj opblanding</Dialog.Title>
			<Dialog.Description>Skriv navnet på den opblanding du vil tilføje</Dialog.Description>
			<div class="grid gap-4 py-4">
				<div class="grid grid-cols-4 items-center gap-4">
					<Label for="other" class="text-right">Opblanding</Label>
					<Input
						id="other"
						placeholder="Noget lækkert"
						class="col-span-3"
						bind:value={otherInput}
						onkeypress={(event: any) => {
							if (event.key == 'Enter') {
								event.preventDefault();
								addOther();
							}
						}}
					/>
				</div>
			</div>
			<Dialog.Footer>
				<Button onclick={addOther}>Tilføj</Button>
			</Dialog.Footer>
		</Dialog.Header>
	</Dialog.Content>
</Dialog.Root>
