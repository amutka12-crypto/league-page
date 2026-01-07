<script>
    import { round } from '$lib/utils/helper'
    import RecordsAndRankings from './RecordsAndRankings.svelte';

    export let key;
    export let leagueManagerRecords;
    export let leagueTeamManagers;
    export let leagueWeekHighs;
    export let leagueWeekLows;
    export let allTimeBiggestBlowouts;
    export let allTimeClosestMatchups;
    export let mostSeasonLongPoints;
    export let leastSeasonLongPoints;
    export let transactionTotals;

    let winPercentages = [];
    let lineupIQs = [];
    let fptsHistories = [];
    let tradesData = [];
    let waiversData = [];
    let showTies = false;

    const setRankingsData = (lRR) => {
        // ⛔ guard against undefined data
        if (!lRR || !transactionTotals?.allTime) return;

        winPercentages = [];
        lineupIQs = [];
        fptsHistories = [];
        tradesData = [];
        waiversData = [];
        showTies = false;

        for (const managerID in lRR) {
            const record = lRR[managerID];
            if (!record) continue;

            const totalGames =
                record.wins + record.ties + record.losses || 1;

            winPercentages.push({
                managerID,
                percentage: round(
                    ((record.wins + record.ties / 2) / totalGames) * 100
                ),
                wins: record.wins,
                ties: record.ties,
                losses: record.losses
            });

            const lineupIQ = {
                managerID,
                fpts: round(record.fptsFor)
            };

            if (record.potentialPoints) {
                lineupIQ.iq = round(
                    (record.fptsFor / record.potentialPoints) * 100
                );
                lineupIQ.potentialPoints = round(record.potentialPoints);
            }

            lineupIQs.push(lineupIQ);

            fptsHistories.push({
                managerID,
                fptsFor: round(record.fptsFor),
                fptsAgainst: round(record.fptsAgainst),
                fptsPerGame: round(record.fptsFor / totalGames)
            });

            if (record.ties > 0) showTies = true;
        }

        for (const managerID in transactionTotals.allTime) {
            const t = transactionTotals.allTime[managerID];
            if (!t) continue;

            tradesData.push({
                managerID,
                trades: t.trade ?? 0
            });

            waiversData.push({
                managerID,
                waivers: t.waiver ?? 0
            });
        }

        winPercentages.sort((a, b) => b.percentage - a.percentage);
        lineupIQs.sort((a, b) => (b.iq ?? 0) - (a.iq ?? 0));
        fptsHistories.sort((a, b) => b.fptsFor - a.fptsFor);
        tradesData.sort((a, b) => b.trades - a.trades);
        waiversData.sort((a, b) => b.waivers - a.waivers);
    };

    $: setRankingsData(leagueManagerRecords);
</script>

{#if leagueManagerRecords && transactionTotals}
    <RecordsAndRankings
        blowouts={allTimeBiggestBlowouts}
        closestMatchups={allTimeClosestMatchups}
        weekRecords={leagueWeekHighs}
        weekLows={leagueWeekLows}
        seasonLongRecords={mostSeasonLongPoints}
        seasonLongLows={leastSeasonLongPoints}
        {showTies}
        {winPercentages}
        {fptsHistories}
        {lineupIQs}
        {tradesData}
        {waiversData}
        prefix="All-Time"
        allTime={true}
        {leagueTeamManagers}
        {key}
    />
{:else}
    <p>Loading records…</p>
{/if}
