<script>
    import { round } from '$lib/utils/helper';
    import { getTeamFromTeamManagers } from '$lib/utils/helperFunctions/universalFunctions';

    export let matchup = [{}, {}];
    export let players = [];
    export let active = null;
    export let ix = 0;
    export let displayWeek = 0;
    export let expandOverride = false;
    export let matchupWeek = null;
    export let leagueTeamManagers = {};
    export let year = leagueTeamManagers?.currentYear ?? new Date().getFullYear();

    let home = matchup?.[0] ?? {};
    let away = matchup?.[1] ?? {};

    let homePointsTotal = 0;
    let homeProjectionTotal = 0;
    let awayPointsTotal = 0;
    let awayProjectionTotal = 0;

    let winning = "home";
    let starters = [];

    const digestStarter = (starter, points) => {
        if (!starter) return {
            name: "Empty",
            avatar: null,
            pos: null,
            team: null,
            opponent: null,
            projection: 0,
            points: 0
        };
        const player = players?.[starter] ?? {};
        const name = player.pos === "DEF" ? player.ln ?? "DEF" : `${player.fn?.[0] ?? ""}. ${player.ln ?? ""}`;
        const projection = player?.wi?.[displayWeek]?.p ? parseFloat(player.wi[displayWeek].p) : 0;
        return {
            name,
            avatar: player.pos === "DEF"
                ? `background-image: url(https://sleepercdn.com/images/team_logos/nfl/${starter.toLowerCase()}.png)`
                : `background-image: url(https://sleepercdn.com/content/nfl/players/thumb/${starter}.jpg), url(https://sleepercdn.com/images/v2/icons/player_default.webp)`,
            pos: player.pos ?? null,
            team: player.t ?? null,
            opponent: player?.wi?.[displayWeek]?.o ?? null,
            projection,
            points: points ?? 0
        };
    };

    const digestStarters = () => {
        home = matchup?.[0] ?? {};
        away = matchup?.[1] ?? {};
        home.manager = getTeamFromTeamManagers(leagueTeamManagers, home.roster_id, year) ?? { name: "", avatar: "" };
        away.manager = getTeamFromTeamManagers(leagueTeamManagers, away.roster_id, year) ?? { name: "", avatar: "" };

        const homeStarters = matchupWeek ? home?.starters?.[matchupWeek] ?? [] : home?.starters ?? [];
        const awayStarters = matchupWeek ? away?.starters?.[matchupWeek] ?? [] : away?.starters ?? [];
        const homePoints = matchupWeek ? home?.points?.[matchupWeek] ?? [] : home?.points ?? [];
        const awayPoints = matchupWeek ? away?.points?.[matchupWeek] ?? [] : away?.points ?? [];

        homePointsTotal = 0;
        homeProjectionTotal = 0;
        awayPointsTotal = 0;
        awayProjectionTotal = 0;

        const localStarters = [];
        for (let i = 0; i < homeStarters.length; i++) {
            const homePoint = homePoints?.[i] ?? 0;
            const awayPoint = awayPoints?.[i] ?? 0;

            const homeObj = digestStarter(homeStarters[i], homePoint);
            const awayObj = digestStarter(awayStarters?.[i], awayPoint);

            homePointsTotal += homePoint;
            awayPointsTotal += awayPoint;
            homeProjectionTotal += homeObj.projection ?? 0;
            awayProjectionTotal += awayObj?.projection ?? 0;

            localStarters.push({ home: homeObj, away: awayObj });
        }

        if (awayPointsTotal < homePointsTotal) winning = "home";
        else if (awayPointsTotal > homePointsTotal) winning = "away";
        else winning = "tied";

        starters = localStarters;
    };

    $: digestStarters();

    let el;
    $: top = el?.getBoundingClientRect()?.top ?? 0;

    const expandClose = () => {
        if (expandOverride) return;
        active = active === ix ? null : ix;
        setTimeout(() => {
            window.scrollTo({ left: 0, top, behavior: 'smooth' });
        }, 200);
    };

    let innerWidth;

    const calcHeight = () => {
        const multiplier = innerWidth < 410 ? 71 : innerWidth < 500 ? 72 : 73;
        const startersLength = matchupWeek ? home?.starters?.[matchupWeek]?.length ?? 0 : home?.starters?.length ?? 0;
        return startersLength * multiplier + 37;
    };
</script>

<svelte:window bind:innerWidth={innerWidth} />
