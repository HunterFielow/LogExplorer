<script lang="ts">
    import { onMount } from 'svelte';
    import { QueryProcessor } from './QueryProcessor';
    import { clickOutside } from './clickOutside';
    let input = '';
    let logclass = false;
    let TableColumn = ['message', 'level', 'Default'];
    let processor = new QueryProcessor();
    let highlightedlog: any;
    let selectedIndex: number = 0;
    let open = false;
    let mouse_x = '0';
    let mouse_y = '0';
    let open_key: string;
    let open_value: string;

    onMount(() => {
        window.addEventListener('message', (event) => {
            const message = event.data;
            switch (message.type) {
                case 'file':
                    processor.formatTextInput(message.text);
                    break;
                case 'config':
                    TableColumn = message.text;
                default:
            }
        });
    });
    tsvscode.postMessage({ type: 'getText', value: '' });
    function openlog(row: any, index: number) {
        highlightedlog = row;
        selectedIndex = index;
        logclass = true;
    }
    function closenav() {
        logclass = false;
    }

    function opendrop(event: any, key: any, value: any) {
        open = true;
        open_key = key;
        open_value = value;
        mouse_x = event.clientX.toString() + 'px';
        mouse_y = event.clientY.toString() + 'px';
    }
    function closedrop() {
        open = false;
    }

     function colorRow(row: any, index: number): string {
        switch(row['level']){
            case 'error': {
                return "rgba(255, 0, 0, 0.2)"
            }
            default: {
                return "";
            }
        }
    }

</script>
<div style="display:flex;height:100vh;overflow-y:hidden;">
    <div id="left-scrollbar" style="flex:7;overflow-y:scroll">
        <div class="input">
            <h3>Query input</h3>
            <input
                type="text"
                bind:value={input}
                on:keypress={(event) => {
                    if (event.key === 'Enter') {
                        processor.query = input;
                        processor.processQuery();
                    }
                }}
            />
        </div>
        <div>
            <table>
                <tr>
                    {#each TableColumn as column}
                        <th>{column}</th>
                    {/each}
                </tr>
                {#each processor.processedObject as row, index}
                    {#if index==selectedIndex}
                    <tr style="background:rgba(0, 255, 0, 0.1)" >
                        {#each TableColumn as column}
                        <td on:click={() => openlog(row, index)}>{JSON.stringify(row[column])}</td>
                        {/each}
                    </tr>
                    {:else}
                    <tr style={"background:"+ colorRow(row, index)} >
                        {#each TableColumn as column}
                        <td on:click={() => openlog(row, index)}>{JSON.stringify(row[column])}</td>
                        {/each}
                    </tr>
                    {/if}
                {/each}
            </table>
        </div>
    </div>
    {#if logclass}
        <div id="right-scrollbar" class="sidenav" style="flex:3;overflow-y:scroll; ">
            {#each Object.keys(highlightedlog) as key}
                <p on:click={(event) => opendrop(event, key, highlightedlog[key])}>
                    {key + ': ' + JSON.stringify(highlightedlog[key], undefined, 4)}
                </p>
            {/each}
        </div>
    {/if}
</div>

{#if open}
    <div
        use:clickOutside
        on:click_outside={closedrop}
        class="dropdown-content"
        style:top={mouse_y}
        style:left={mouse_x}
    >
        <menu class="dropdown-content">
            <li
                class="menu-item"
                on:click={() => {
                    processor.addCondition(open_key, open_value, '=');
                    input = processor.query;
                    closedrop();
                    processor.processedObject = processor.processedObject;
                }}
            >
                Search only value
            </li>
            <li
                class="menu-item"
                on:click={() => {
                    processor.addCondition(open_key, open_value, '!=');
                    input = processor.query;
                    closedrop();
                    processor.processedObject = processor.processedObject;
                }}
            >
                Exclude value
            </li>
            <li
                class="menu-item"
                on:click={() => {
                    processor.addCondition(open_key, open_value, '>');
                    input = processor.query;
                    closedrop();
                    processor.processedObject = processor.processedObject;
                }}
            >
                Greater than value
            </li>
            <li
                class="menu-item"
                on:click={() => {
                    processor.addCondition(open_key, open_value, '>=');
                    input = processor.query;
                    closedrop();
                    processor.processedObject = processor.processedObject;
                }}
            >
                Greater or equal than value
            </li>
            <li
                class="menu-item"
                on:click={() => {
                    processor.addCondition(open_key, open_value, '<');
                    input = processor.query;
                    closedrop();
                    processor.processedObject = processor.processedObject;
                }}
            >
                Less than value
            </li>
            <li
                class="menu-item"
                on:click={() => {
                    processor.addCondition(open_key, open_value, '<=');
                    input = processor.query;
                    closedrop();
                    processor.processedObject = processor.processedObject;
                }}
            >
                Less or equal than
            </li>
            <li class="menu-break" style="color:gray">⸺⸺⸺⸺⸺⸺⸺</li>
            <li class="menu-item">Inspect element (not working)</li>
        </menu>
    </div>
{/if}
