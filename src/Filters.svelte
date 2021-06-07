<script>
    import { slide } from "svelte/transition";

    export let filters = [];
    export let selectedFilters = [];
    export let displayFiltersOnOpen = false;

    let showFilters = displayFiltersOnOpen;
    let filterDetails = { show: false, data: {} };

    function toggleFilterDetails(filter) {
        if (filterDetails.show && filterDetails.data.key == filter.key) {
            filterDetails.show = false;
            filterDetails.data = {};
        } else {
            filterDetails.show = true;
            filterDetails.data = {
                type: filter.type,
                key: filter.key,
                label: filter.label,
                values: filter.values
            };
        }
    }

    function triggerFilter(e, filter, value = null) {
        e.stopPropagation();
        e.preventDefault();

        switch (filter.type) {
            case 'checkbox':
                if (!value) {
                    return $selectedFilters[filter.key] = $selectedFilters[filter.key] == false ? null : !$selectedFilters[filter.key];
                }

                if ($selectedFilters[filter.key]) {
                    if ($selectedFilters[filter.key].find((v) => v == value)) {
                        return $selectedFilters[filter.key] = $selectedFilters[filter.key].filter((v) => v != value);
                    }

                    return $selectedFilters[filter.key] = [...$selectedFilters[filter.key], value];
                } 

                return $selectedFilters[filter.key] = [value];
            case 'multi-checkbox':
            case 'daterange':
            case 'range':
                return toggleFilterDetails(filter);
        }
    }

    $: isSelected = (key) => {
        if (Array.isArray($selectedFilters[key]))
            return $selectedFilters[key].length;
        else if ($selectedFilters[key] && $selectedFilters[key].hasOwnProperty('from') && $selectedFilters[key].hasOwnProperty('to'))
            return $selectedFilters[key]['from'] !== null || $selectedFilters[key]['to'] !== null;
        else
            return $selectedFilters[key] !== null;
    }
</script>


{#if filters.length > 0}
    <div class="filters-container">
        <div class="filters-show-button" on:click={(e) => { e.stopPropagation(); showFilters = !showFilters }}>
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"/>
            </svg>
            {#if showFilters}
                <span>Hide filters</span>
            {:else}
                <span>Display filters</span>
            {/if}
        </div>
        {#if showFilters}
            <div transition:slide class="filters">
                {#each filters as filter}
                    <div class="filter-item" class:filter-item-details-opened={filterDetails && filterDetails.data.key == filter.key} on:click={(e) => triggerFilter(e, filter)} class:filter-item-selected={isSelected(filter.key)}>
                        {#if filter.type == "multi-checkbox" && isSelected(filter.key)}
                            {filter.label.toLowerCase()} : <span>{$selectedFilters[filter.key][0]} {$selectedFilters[filter.key].length > 1 ? `(+${$selectedFilters[filter.key].length - 1})` : ''}</span>
                        {:else if filter.type == "checkbox" && isSelected(filter.key)}
                            {filter.label.toLowerCase()} : <span>
                                {#if $selectedFilters[filter.key]}
                                    &#10003; <!-- Checkmark -->
                                {:else}
                                    &#215; <!-- Times -->
                                {/if}
                            </span>
                        {:else if (filter.type == "range" || filter.type == "daterange") && isSelected(filter.key)}
                            {filter.label.toLowerCase()} : 
                            {#if $selectedFilters[filter.key]["from"]}
                                <span>&#8805 {$selectedFilters[filter.key]["from"]}</span> <!-- >= -->
                            {/if}
                            {#if $selectedFilters[filter.key]["to"]}
                                <span>&#8804 {$selectedFilters[filter.key]["to"]}</span> <!-- <= -->
                            {/if}
                        {:else}
                            {filter.label}
                        {/if}
                    </div>
                {/each}
            </div>
            {#if filterDetails.show}
                <div class="filter-details" transition:slide>
                    {#if filterDetails.data.type == "range"}
                        <div class="filter-details-range">
                            <div>&#8805 <input type="number" bind:value={$selectedFilters[filterDetails.data.key].from } /></div>
                            <div>&#8804 <input type="number" bind:value={$selectedFilters[filterDetails.data.key].to } /></div>
                        </div>
                    {:else if filterDetails.data.type == "daterange"}
                        <div class="filter-details-daterange">
                            <div>&#8805  <input type="date" bind:value={$selectedFilters[filterDetails.data.key].from } /></div>
                            <div>&#8804  <input type="date" bind:value={$selectedFilters[filterDetails.data.key].to } /></div>
                        </div>
                    {:else if filterDetails.data.type == "multi-checkbox"}
                        {#each filterDetails.data.values as value}
                            <label on:click={(e) => triggerFilter(e, { key: filterDetails.data.key, type: "checkbox" }, value)}>
                                <input type="checkbox" {value} bind:group={$selectedFilters[filterDetails.data.key]} />
                                {value}
                            </label>
                        {/each}
                    {/if}
                </div>
            {/if}
        {/if}
    </div>
{/if}

<style>
    .filters-container {
        border-bottom: 1px solid #dedede;
    }

    .filters-container .filters-show-button {
        display: flex;
        align-items: center;
        padding: var(--groupTitlePadding, 7px 20px);
        cursor: pointer;
        color: var(--itemIsActiveBG, #007aff);
    }

    .filters-container .filters-show-button svg {
        stroke: var(--itemIsActiveBG, #007aff);
        margin-right: .3rem;
        width: 15px;
    }

    .filters {
        padding: var(--groupTitlePadding, 7px 20px);
        display: flex;
        flex-wrap: wrap;
    }

    .filters .filter-item {
        cursor: pointer;
        border: 1px solid #333;
        border-radius: 50px;
        padding: .3em 1.2em;
        margin: 0.2rem;
    }

    .filters .filter-item span {
        font-weight: 600;
    }
    
    .filters .filter-item-selected {
        border: 1px solid var(--itemIsActiveBG, #007aff);
        background: transparent;
        color: var(--itemIsActiveBG, #007aff);
    }

    .filters .filter-item-details-opened {
        border: none;
        background: var(--itemIsActiveBG, #007aff);
        color: #fff;
    }

    .filter-details {
        padding: var(--groupTitlePadding, 2px 20px);
    }

    .filter-details label {
        cursor: pointer;
    }

    .filter-details-range div,
    .filter-details-daterange div {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
</style>