<script lang="ts">
	import * as Card from '$lib/components/ui/card/index.js';
	import * as Dialog from '$lib/components/ui/dialog/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import { Separator } from '$lib/components/ui/separator/index.js';
	import { Label } from '$lib/components/ui/label';
	import { Input } from '$lib/components/ui/input';
	import { Trash2 } from 'lucide-svelte';

	let alcoholics: string[] = $state([]);
	let alcoholDialogOpen: boolean = $state(false);
	let alcoholInput: string = $state('');

	function addAlcohol() {
		if (alcoholics.includes(alcoholInput)) {
			return;
		}
		alcoholics.push(alcoholInput);
		alcoholInput = '';
		alcoholDialogOpen = false;
	}

	function removeAlcohol(alcohol: string) {
		alcoholics = alcoholics.filter((it) => it !== alcohol);
	}
</script>

<div class="grid h-screen w-screen grid-cols-2 grid-rows-2">
	<div class="col-span-2 place-content-center justify-items-center bg-blue-500">
		<Card.Root class="min-w-xl w-1/2">
			<Card.Header>
				<Card.Title>DrinkGPT</Card.Title>
				<Card.Description>Angiv lager forneden, tryk på knappen og lad magien ske</Card.Description>
			</Card.Header>
			<Card.Content>
				<p>Card Content</p>
			</Card.Content>
			<Card.Footer>
				<p>Card Footer</p>
			</Card.Footer>
		</Card.Root>
	</div>
	<div class="justify-items-center bg-green-500 p-16">
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
						<Card.Root class="flex w-full justify-between p-6">
							<Card.Title class="mt-2">{alcohol}</Card.Title>

							<Button onclick={(_) => removeAlcohol(alcohol)} variant="outline" size="icon">
								<Trash2 />
								<span class="sr-only">Remove {alcohol}</span>
							</Button>
						</Card.Root>
					{/each}
				{:else}
					<p>Add some alcohol to begin...</p>
				{/if}
			</Card.Content>
		</Card.Root>
	</div>
	<div class="justify-items-center bg-green-500 p-16">
		<Card.Root class="w-full">
			<Card.Header>
				<Card.Title>Opblanding</Card.Title>
				<Card.Description>Angiv hvilke slags opblandinger, der er på lager</Card.Description>
			</Card.Header>
			<Card.Content>
				<p>Card Content</p>
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
