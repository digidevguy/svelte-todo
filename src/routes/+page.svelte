<script lang="ts">
	import Button from '@/components/ui/button/button.svelte';
	import { Checkbox } from '@/components/ui/checkbox';
	import { Input } from '@/components/ui/input';
	import { Label } from '@/components/ui/label';
	import * as Select from '@/components/ui/select';
	import { fade } from 'svelte/transition';
	import Checkmark from '@/svgs/checkmark.svelte';
	import * as Card from '@/components/ui/card';
	import { Separator } from '@/components/ui/separator';

	type Priority = {
		value: 'Urgent' | 'High' | 'Medium' | 'Low';
	};
	type Filter = {
		value: 'All' | 'Urgent' | 'High' | 'Medium' | 'Low';
	};

	type Todo = {
		text: string;
		priority: Priority;
		done: boolean;
	};

	let priorities: Priority[] = [
		{ value: 'Urgent' },
		{ value: 'High' },
		{ value: 'Medium' },
		{ value: 'Low' }
	];
	let filterOptions: Filter[] = [
		{ value: 'All' },
		{ value: 'Urgent' },
		{ value: 'High' },
		{ value: 'Medium' },
		{ value: 'Low' }
	];

	let priorityInput = $state<Priority>({ value: 'Urgent' });
	let priorityFilter = $state<Filter>({ value: 'All' });

	let todos = $state<Todo[]>([]);
	let filteredTodos = $derived<Todo[]>(filterTodos());
	let ongoingTotal = $state(0);

	let todoInput = $state<string>('');

	function addTodo(e: KeyboardEvent) {
		if (e.key !== 'Enter') return;

		todos.push({
			text: todoInput,
			priority: priorityInput,
			done: false
		});

		todoInput = '';
		priorityInput = { value: 'Low' };
	}

	function editTodo(e: Event) {
		const inputEl = e.target as HTMLInputElement;
		const index = +inputEl.dataset.index!;
		todos[index].text = inputEl.value;
	}

	function toggleDone(i: number) {
		todos[i].done = !todos[i].done;
	}

	function filterTodos() {
		if (priorityFilter.value === 'All') return todos;

		return todos.filter((todo) => todo.priority.value === priorityFilter.value);
	}

	function clearCompleted() {
		ongoingTotal += todos.filter((todo) => todo.done === true).length;
		todos = todos.filter((todo) => todo.done !== true);
	}

	$effect(() => {
		const savedTodos = localStorage.getItem('todos');
		const savedTotal = localStorage.getItem('total');
		if (savedTodos) todos = JSON.parse(savedTodos);
		if (savedTotal) ongoingTotal = Number(savedTotal);
	});

	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos));
	});

	$effect(() => {
		localStorage.setItem('total', String(ongoingTotal));
	});
</script>

<div class="mx-auto flex w-full max-w-md flex-col gap-4 px-4 py-2">
	<div class="flex flex-col items-center gap-4 rounded-md border border-orange-600 p-4">
		<div class="flex items-center gap-2">
			<h1 class="font-raleway font-semibold">Welcome to Local Todos</h1>
			<Checkmark />
		</div>
		<Separator class="max-w-xs bg-orange-700" />
		<div class="flex w-full flex-col gap-4">
			<div class="flex w-full items-center justify-center gap-4">
				<Label for="priority-setter">Priority</Label>
				<Select.Root bind:selected={priorityInput}>
					<Select.Trigger>
						<Select.Value placeholder="Urgent" id="priority-setter" />
					</Select.Trigger>
					<Select.Content>
						{#each priorities as priority}
							<Select.Item value={priority.value}>{priority.value}</Select.Item>
						{/each}
					</Select.Content>
				</Select.Root>
			</div>
			<Input
				type="text"
				class="w-full"
				bind:value={todoInput}
				placeholder="I want to. . ."
				onkeydown={addTodo}
			/>
		</div>
	</div>

	<div class="flex items-center gap-4">
		<Label for="filter-select" class="shrink-0">Filter by:</Label>
		<Select.Root bind:selected={priorityFilter}>
			<Select.Trigger>
				<Select.Value placeholder="All" id="filter-select" />
			</Select.Trigger>
			<Select.Content>
				{#each filterOptions as { value }}
					<Select.Item {value}>{value}</Select.Item>
				{/each}
			</Select.Content>
		</Select.Root>
	</div>
	<Separator class="mx-auto max-w-xs bg-orange-700" />
	<div id="todo-list" class="flex flex-col gap-4">
		{#each filteredTodos as { text, done }, i}
			<div
				class="relative transition-opacity duration-[0.3] ease-in-out"
				class:completed={done}
				transition:fade={{ duration: 200 }}
			>
				<Input
					type="text"
					value={text}
					onchange={editTodo}
					data-index={i}
					class="w-full p-4"
					disabled={done}
				/>
				<Checkbox
					data-index={i}
					checked={done}
					onCheckedChange={() => toggleDone(i)}
					class="absolute right-[4%] top-1/2 translate-y-[-50%]"
				/>
			</div>
		{:else}
			<p class="text-center">This list is currently empty</p>
		{/each}
	</div>
	{#if todos.length > 0}
		<Button class="mx-auto" variant="outline" onclick={clearCompleted}>Clear completed tasks</Button
		>
	{/if}
	{#if ongoingTotal === 0}
		<p class="text-center">No completed tasks yet</p>
	{:else}
		<p class="text-center">{ongoingTotal} completed tasks</p>
	{/if}
</div>

<style>
	:global(body) {
		padding-top: 1rem;
		min-height: 100vh;
	}

	.completed {
		opacity: 0.4;
	}
</style>
