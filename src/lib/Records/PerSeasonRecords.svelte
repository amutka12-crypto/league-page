<script>
    import Button, { Group, Label } from '@smui/button';
    import { round } from '$lib/utils/helper';
    import RecordsAndRankings from './RecordsAndRankings.svelte';

    export let leagueRosterRecords;
    export let seasonWeekRecords;
    export let leagueTeamManagers;
    export let currentYear;
    export let lastYear;
    export let transactionTotals;
    export let key;

    let yearsObj = {};
    let years = [];
    let display = 0;

    const setData = (lRR) => {
        // ⛔ hard guard
        if (
            !lRR ||
            !seasonWeekRecords ||
            !transactionTotals?.seasons ||
            currentYear == null ||
            lastYear == null
        ) {
            years = [];
            return;
        }

        yearsObj = {};
        years = [];

        for (let y = currentYear; y >= lastYear; y--) {
            yearsObj[y] = {
                year: y,
                seasonLongRecords: [],
                seasonLongLows: [],
                winPercentages: [],
                lineupIQs: [],
                fptsHistories: [],
                tradesData: [],
                waiversData: [],
                blowouts: [],
                closestMatchups: [],
                showTies: false
            };
        }

        for (const rec of seasonWeekRecords) {
            if (!yearsObj[rec.year]) continue;
            yearsObj[rec.year].weekRecords = rec.seasonPointsHighs;
            yearsObj[rec.year].weekLows = rec.seasonPointsLows;
            yearsObj[rec.year].blowouts = rec.biggestBlowouts;
            yearsObj[rec.year].closestMatchups = rec.closestMatchups;
        }

        for (const season in transactionTotals.seasons) {
            if (!yearsObj[season]) continue;
            for (const rosterID in transactionTotals.seasons[season]) {
                const t = transactionTotals.seasons[season][rosterID];
                if (!t) continue;

                yearsObj[season].tradesData.push({
                    rosterID,
                    trades: t.trade ?? 0
                });

                yearsObj[season].waiversData.push({
                    rosterID,
                    waivers: t.waiver ?? 0
                });
            }
        }

        for (const rosterID in lRR) {
            const record = lRR[rosterID];
            if (!record?.years) continue;

            for (const season of record.years) {
                if (!yearsObj[season.year]) continue;

                if (season.ties > 0) yearsObj[season.year].showTies = true;

                yearsObj[season.year].seasonLongRecords.push({
                    rosterID,
                    fpts: round(season.fpts),
