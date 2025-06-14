<script lang="ts">
    import { writable } from "svelte/store";

    export let folders = writable([
        {
            id: 1,
            name: "Moza",
            notes: [
                { id: 1, type: "simple", content: "Moza times" },
                {
                    id: 2,
                    type: "table",
                    table: {
                        columns: ["Vilius", "Viliaus enemy"],
                        rows: [["1.52", "1.51"]],
                    },
                },
            ],
        },
        {
            id: 2,
            name: "SPA",
            notes: [
                { id: 1, type: "simple", content: "SPA times" },
                {
                    id: 2,
                    type: "table",
                    table: {
                        columns: ["Vilius", "Viliaus enemy"],
                        rows: [["2:26", "2:25"]],
                    },
                },
            ],
        }
    ],
);

    let newFolderName = "";
    let searchTerm = "";
    let newNoteContent = "";
    let editingNoteId: number | null = null;
    let editingCell: { noteId: number, rowIndex: number, colIndex: number } | null = null;

    function editNote(folderId: number, noteId: number, newContent: string) {
        folders.update((current) =>
            current.map((folder) =>
                folder.id === folderId
                    ? {
                          ...folder,
                          notes: folder.notes.map((note) => {
                              if (note.id === noteId && note.type === "simple") {
                                  return {
                                      id: note.id,
                                      type: note.type,
                                      content: newContent
                                  };
                              }
                              return note;
                          }),
                      }
                    : folder
            )
        );
        editingNoteId = null;
    }

    function editTableCell(folderId: number, noteId: number, rowIndex: number, colIndex: number, newValue: string) {
        folders.update((current) =>
            current.map((folder) =>
                folder.id === folderId
                    ? {
                          ...folder,
                          notes: folder.notes.map((note) => {
                              if (note.id === noteId && note.type === "table" && note.table) {
                                  const newRows = [...note.table.rows];
                                  newRows[rowIndex] = [...newRows[rowIndex]];
                                  newRows[rowIndex][colIndex] = newValue;
                                  return {
                                      ...note,
                                      table: {
                                          ...note.table,
                                          rows: newRows
                                      }
                                  };
                              }
                              return note;
                          }),
                      }
                    : folder
            )
        );
        editingCell = null;
    }

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
            notes: folder.notes.filter((note) => {
                if (note.type === "simple") {
                    return note.content?.toLowerCase().includes(searchTerm.toLowerCase()) ?? false;
                } else if (note.type === "table" && note.table) {
                    // Search in table columns and rows
                    const columnsMatch = note.table.columns.some(col => 
                        col.toLowerCase().includes(searchTerm.toLowerCase())
                    );
                    const rowsMatch = note.table.rows.some(row =>
                        row.some(cell => cell.toLowerCase().includes(searchTerm.toLowerCase()))
                    );
                    return columnsMatch || rowsMatch;
                }
                return false;
            }),
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
                                {#if editingNoteId === note.id}
                                    <input
                                        type="text"
                                        value={note.content}
                                        class="w-full p-1 border rounded"
                                        on:keydown={(e) => {
                                            if (e.key === 'Enter') {
                                                editNote(folder.id, note.id, (e.target as HTMLInputElement).value);
                                            } else if (e.key === 'Escape') {
                                                editingNoteId = null;
                                            }
                                        }}
                                    />
                                {:else}
                                    <div 
                                        role="button"
                                        tabindex="0"
                                        on:dblclick={() => editingNoteId = note.id}
                                        on:keydown={(e) => {
                                            if (e.key === 'Enter') {
                                                editingNoteId = note.id;
                                            }
                                        }}
                                    >
                                        {note.content}
                                    </div>
                                {/if}
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
                                            {#each note.table.rows as row, row_index}
                                                <tr>
                                                    {#each row as cell, cell_index}
                                                        <td
                                                            class="px-4 py-2 border-gray-300 border-t border-b"
                                                        >
                                                            {#if editingCell && editingCell.noteId === note.id && editingCell.rowIndex === row_index && editingCell.colIndex === cell_index}
                                                                <input
                                                                    type="text"
                                                                    value={cell}
                                                                    class="w-full p-1 border rounded"
                                                                    on:keydown={(e) => {
                                                                        if (e.key === 'Enter') {
                                                                            editTableCell(folder.id, note.id, row_index, cell_index, (e.target as HTMLInputElement).value);
                                                                        } else if (e.key === 'Escape') {
                                                                            editingCell = null;
                                                                        }
                                                                    }}
                                                                />
                                                            {:else}
                                                                <div 
                                                                    role="button"
                                                                    tabindex="0"
                                                                    on:dblclick={() => editingCell = { noteId: note.id, rowIndex: row_index, colIndex: cell_index }}
                                                                    on:keydown={(e) => {
                                                                        if (e.key === 'Enter') {
                                                                            editingCell = { noteId: note.id, rowIndex: row_index, colIndex: cell_index };
                                                                        }
                                                                    }}
                                                                >
                                                                    {cell}
                                                                </div>
                                                            {/if}
                                                        </td>
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
