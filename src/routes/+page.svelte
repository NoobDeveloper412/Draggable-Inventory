<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { writable } from 'svelte/store';
    import { browser } from '$app/environment';

    let isVisible = writable(true);

    let items = writable<Item[]>([
        { name: 'item1', label: 'Item 1', count: 10 },
        { name: 'item2', label: 'Item 2', count: 5 },
        { name: 'item3', label: 'Item 3', count: 20 },
        { name: 'item4', label: 'Item 4', count: 8 },
        { name: 'item5', label: 'Item 5', count: 15 },
        { name: 'item6', label: 'Item 6', count: 12 }
    ]);

    let droppedItems = writable<Item[]>([]);

    interface Item {
        name: string;
        label: string;
        count: number;
    }

    async function hideUI(): Promise<void> {
        isVisible.set(false);
        await new Promise<void>(resolve => setTimeout(resolve, 300));
    }

    function handleKeydown(event: KeyboardEvent) {
        if (event.key === 'Escape') {
            event.preventDefault();
            hideUI();
        }
    }

    if (browser) {
        onMount(() => {
            window.addEventListener('keydown', handleKeydown);
        });

        onDestroy(() => {
            window.removeEventListener('keydown', handleKeydown);
        });
    }

    function handleDragStart(event: DragEvent, itemIndex: number, origin: 'items' | 'droppedItems') {
        event.dataTransfer?.setData('itemIndex', itemIndex.toString());
        event.dataTransfer?.setData('origin', origin);
    }

    function handleDrop(event: DragEvent, target: 'items' | 'droppedItems') {
        const itemIndex = event.dataTransfer?.getData('itemIndex');
        const origin = event.dataTransfer?.getData('origin');
        if (itemIndex !== null && origin !== null) {
            const index = parseInt(itemIndex);
            if (origin === 'items' && target === 'droppedItems') {
                items.update(itemsArray => {
                    const [item] = itemsArray.splice(index, 1);
                    droppedItems.update(droppedArray => [...droppedArray, item]);
                    return itemsArray;
                });
            } else if (origin === 'droppedItems' && target === 'items') {
                droppedItems.update(droppedArray => {
                    const [item] = droppedArray.splice(index, 1);
                    items.update(itemsArray => [...itemsArray, item]);
                    return droppedArray;
                });
            }
        }
    }

    function handleDragOver(event: DragEvent) {
        event.preventDefault();
    }
</script>

<div class="h-full px-20"> 
    {#if $isVisible}
        <div class="flex items-center justify-between h-screen p-6 rounded-lg gap-8">
            <div class="w-1/2 bg-[#596475] border border-2 rounded-sm">
                <h2 class="text-2xl text-white capitalize font-mono font-bold text-center mb-4">Main Items</h2>
                <div class="grid grid-cols-3 gap-4 p-4 rounded-lg" on:dragover={handleDragOver} on:drop={(e) => handleDrop(e, 'items')}>
                    {#each $items as item, itemIndex (itemIndex)}
                        <button
                            class="grid-item p-4 bg-[#596475] border cursor-pointer border-white rounded-lg shadow-md hover:bg-gray-200 transition-all"
                            draggable="true"
                            on:dragstart={(e) => handleDragStart(e, itemIndex, 'items')}
                        >
                        <div class="py-4">
                            <img class="w-16 h-16 mx-auto mb-2 object-cover" src={`/images/images.jpeg`} alt={item.label} />
                        </div>
                            <span class="block text-center font-bold text-gray-700">{item.label}</span>
                            <span class="block text-center text-sm text-gray-500">x{item.count}</span>
                        </button>
                    {/each}
                </div>
            </div>

            <div class="w-1/2 bg-[#596475] border border-2 rounded-sm">
                <h2 class="text-2xl text-white capitalize font-mono font-bold text-center mb-4">Dropped Items</h2>
                <div class="grid grid-cols-3 gap-4 p-4 rounded-lg" on:dragover={handleDragOver} on:drop={(e) => handleDrop(e, 'droppedItems')}>
                    {#each $droppedItems as item, itemIndex (itemIndex)}
                        <button
                            class="grid-item p-4 bg-gray-100 border border-white rounded-lg shadow-md hover:bg-gray-200 transition-all cursor-pointer"
                            draggable="true"
                            on:dragstart={(e) => handleDragStart(e, itemIndex, 'droppedItems')}
                        >
                            <img class="w-16 h-16 mx-auto mb-2 object-cover" src={`/images/images.jpeg`} alt={item.label} />
                            <span class="block text-center font-bold text-gray-700">{item.label}</span>
                            <span class="block text-center text-sm text-gray-500">x{item.count}</span>
                        </button>
                    {/each}
                </div>
            </div>
        </div>
    {/if}
</div>

<style>
    .grid-item.hovering {
        outline: 2px solid blue;
    }
</style>
