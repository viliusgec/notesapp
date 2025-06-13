<script lang="ts">
    import { writable } from "svelte/store";

    export let folders = writable([
        {
            id: 1,
            name: "General",
            notes: [
                { id: 1, type: "simple", content: "This is a simple note" },
                {
                    id: 2,
                    type: "table",
                    table: {
                        columns: ["Column 1", "Column 2"],
                        rows: [["Row 1-1", "Row 1-2"]],
                    },
                },
            ],
        },
    ]);

    let newFolderName = "";
    let searchTerm = "";
    let newNoteContent = "";

    function addFolder() {
        folders.update((current) => [
            ...current,
            { id: Date.now(), name: newFolderName, notes: [] },
        ]);
        newFolderName = "";
    }

    function deleteFolder(id: number) {
        folders.update((current) =>
            current.filter((folder) => folder.id !== id),
        );
    }

    function renameFolder(id: number, newName: string) {
        folders.update((current) =>
            current.map((folder) =>
                folder.id === id ? { ...folder, name: newName } : folder,
            ),
        );
    }

    function addNoteToFolder(id: number, noteContent: string) {
        if (!noteContent.trim()) return;
        folders.update((current) =>
            current.map((folder) =>
                folder.id === id
                    ? {
                          ...folder,
                          notes: [
                              ...folder.notes,
                              {
                                  id: Date.now(),
                                  type: "simple",
                                  content: noteContent,
                              },
                          ],
                      }
                    : folder,
            ),
        );
        newNoteContent = "";
    }

    // New function to delete a note from a folder
    function deleteNote(folderId: number, noteId: number) {
        folders.update((current) =>
            current.map((folder) =>
                folder.id === folderId
                    ? {
                          ...folder,
                          notes: folder.notes.filter((note) => note.id !== noteId),
                      }
                    : folder,
            ),
        );
    }

    $: filteredFolders = $folders.map((folder) => {
        return {
            ...folder,
            notes: folder.notes.filter((note) =>
                note.content?.toLowerCase().includes(searchTerm.toLowerCase()),
            ),
        };
    });
</script>

<h1>Notes App</h1>

<h2>Folders</h2>
<input
    bind:value={searchTerm}
    placeholder="Search notes"
    class="p-2 border rounded-md mr-2"
/>
<ul class="bg-gray-100 p-4 rounded-md shadow-md space-y-4">
    {#each filteredFolders as folder}
        <li class="bg-gray-50 p-4 rounded-md shadow-md">
            <h3 class="text-lg font-semibold mb-2">{folder.name}</h3>
            <button
                on:click={() => deleteFolder(folder.id)}
                class="bg-red-500 text-gray-50 px-4 py-2 mr-2 rounded-md hover:bg-red-600"
                >Delete Folder</button
            >
            <input
                type="text"
                placeholder="Rename folder"
                on:keydown={(e) =>
                    e.key === "Enter" &&
                    e.target &&
                    renameFolder(
                        folder.id,
                        (e.target as HTMLInputElement).value,
                    )}
                class="p-2 border rounded-md mr-2"
            />
            <ul class="bg-gray-200 p-4 rounded-md space-y-2 mt-4">
                {#each folder.notes as note}
                    <li class="bg-gray-100 p-2 rounded-md shadow-md flex justify-between items-center">
                        <div class="flex-1">
                            {#if note.type == "simple"}
                                {note.content}
                            {:else}
                                <table class="min-w-full divide-y divide-gray-200">
                                    <thead>
                                        <tr>
                                            {#if note.table}
                                                {#each note.table.columns as column}
                                                    <th
                                                        class="px-4 py-2 font-semibold text-gray-700 border-gray-300 border-t border-b"
                                                        >{column}</th
                                                    >
                                                {/each}
                                            {/if}
                                        </tr>
                                    </thead>
                                    <tbody>
                                        {#if note.table}
                                            {#each note.table.rows as row}
                                                <tr>
                                                    {#each row as cell}
                                                        <td
                                                            class="px-4 py-2 border-gray-300 border-t border-b"
                                                            >{cell}</td
                                                        >
                                                    {/each}
                                                </tr>
                                            {/each}
                                        {/if}
                                    </tbody>
                                </table>
                            {/if}
                        </div>
                        <button
                            on:click={() => deleteNote(folder.id, note.id)}
                            class="ml-4 bg-red-500 text-gray-50 px-3 py-1 rounded-md hover:bg-red-600"
                            >Delete Note</button
                        >
                    </li>
                {/each}
            </ul>
            <input
                bind:value={newNoteContent}
                type="text"
                placeholder="Add new note"
                class="p-2 border rounded-md mr-2 mt-4"
            />
            <button
                on:click={() => addNoteToFolder(folder.id, newNoteContent)}
                class="bg-blue-500 text-gray-50 px-4 py-2 rounded-md hover:bg-blue-600 mt-4"
                >Add Note</button
            >
        </li>
    {/each}
</ul>

<input bind:value={newFolderName} /><button on:click={addFolder}>
    Add Folder</button
>
