<script>
    import LinearProgress from '@smui/linear-progress';
    import MatchupWeeks from './MatchupWeeks.svelte';
    import Brackets from './Brackets.svelte';
    import Button, { Group, Label } from '@smui/button';
    import { goto } from '$app/navigation';
    import { onMount } from 'svelte';
    import { loadPlayers } from '$lib/utils/helper';

    export let queryWeek, leagueTeamManagersData, matchupsData, bracketsData, playersData;

    let players = [], matchupWeeks = [], year, week, regularSeasonLength, brackets = {}, leagueTeamManagers;

    let loading = true;
    let selection = 'regular';

    onMount(async () => {
        brackets = await bracketsData ?? {};
        const matchupsInfo = await matchupsData ?? {};
        leagueTeamManagers = await leagueTeamManagersData ?? {};
        matchupWeeks = matchupsInfo.matchupWeeks ?? [];
        year = matchupsInfo.year ?? new Date().getFullYear();
        week = matchupsInfo.week ?? 1;
        regularSeasonLength = matchupsInfo.regularSeasonLength ?? 14;

        const playersInfo = await playersData ?? {};
        players = playersInfo.players ?? [];
        loading = false;

        if(playersInfo.stale) {
            const newPlayersInfo = await loadPlayers(null, true);
            players = newPlayersInfo.players ?? [];
        }
    });

    const changeSelection = (s) => {
        if(s === 'regular') {
            queryWeek = 1;
            goto(`/matchups?week=1`, {noscroll: true});
        } else if(selection === 'regular') {
            queryWeek = 99; // placeholder for playoffs start
            goto(`/matchups?week=99`, {noscroll: true});
        }
        selection = s;
    }
</script>

<style>
    .message {
        display: block;
        width: 85%;
        max-width: 500px;
        margin: 80px auto;
    }

    .buttonHolder {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin: 3em 0;
    }
</style>

{#if loading}
    <div class="message">
        <p>Loading league matchups...</p>
        <LinearProgress indeterminate />
    </div>
{:else}
    {#if matchupWeeks?.length}
        <div class="buttonHolder">
            <Group variant="outlined">
                <Button class="selectionButtons" onclick={() => changeSelection('regular')} variant={selection === 'regular' ? "raised" : "outlined"}>
                    <Label>Regular Season</Label>
                </Button>
                <Button class="selectionButtons" onclick={() => changeSelection('champions')} variant={selection === 'champions' || selection === 'losers' ? "raised" : "outlined"}>
                    <Label>Playoffs</Label>
                </Button>
            </Group>

            {#if selection === 'champions' || selection === 'losers'}
                <Group variant="outlined">
                    <Button class="selectionButtons" onclick={() => changeSelection('champions')} variant={selection === 'champions' ? "raised" : "outlined"}>
                        <Label>Champions' Bracket</Label>
                    </Button>
                    <Button class="selectionButtons" onclick={() => changeSelection('losers')} variant={selection === 'losers' ? "raised" : "outlined"}>
                        <Label>Losers' Bracket</Label>
                    </Button>
                </Group>
            {/if}
        </div>

        {#if selection === 'regular'}
            <MatchupWeeks 
                {players} 
                {queryWeek} 
                {matchupWeeks} 
                {regularSeasonLength} 
                {year} 
                {week} 
                bind:selection={selection} 
                {leagueTeamManagers} />
        {/if}
    {:else}
        <div class="message">
            <p>No upcoming matchups...</p>
        </div>
    {/if}

    {#if brackets?.champs?.bracket?.[0]?.[0]?.[0]?.points && (selection === 'champions' || selection === 'losers')}
        <Brackets 
            {queryWeek} 
            {leagueTeamManagers} 
            {players} 
            {brackets} 
            bind:selection={selection} />
    {/if}
{/if}
