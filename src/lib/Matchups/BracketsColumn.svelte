<script>
    import { round } from "$lib/utils/helper";
    import { getAvatarFromTeamManagers, getTeamNameFromTeamManagers } from "$lib/utils/helperFunctions/universalFunctions";

    export let leagueTeamManagers = {};
    export let players = [];
    export let matchCol = [];
    export let playoffsStart = 1;
    export let ix = 0;
    export let playoffLength = 1;
    export let consolation = false;
    export let losers = false;
    export let numRosters = 0;
    export let consolationNum = 0;
    export let selected = null;

    let label = '';

    const setLabel = () => {
        if ((matchCol?.length ?? 0) > 1) {
            switch (playoffLength - ix) {
                case 1:
                    label = losers ? 'Toilet Bowl' : 'Championship Match';
                    break;
                case 2:
                    label = 'Semifinals';
                    break;
                case 3:
                    label = 'Quarterfinals';
                    break;
                case 4:
                    label = 'Eighth-Finals';
                    break;
                default:
                    label = '';
            }
        } else {
            if (!consolation) {
                label = losers ? 'Toilet Bowl' : 'Championship Match';
                return;
            }
            label = losers
                ? nThPlace(numRosters - 2 * (consolationNum + 1))
                : nThPlace(1 + 2 * (consolationNum + 1));
        }
    };

    const nThPlace = (num) => {
        let end = 'th';
        if (num % 10 === 1 && num % 100 !== 11) end = 'st';
        else if (num % 10 === 2 && num % 100 !== 12) end = 'nd';
        else if (num % 10 === 3 && num % 100 !== 13) end = 'rd';
        return `${num}${end} Place`;
    };

    $: setLabel();

    let anchors = {};
    let drawBracket = false;

    const setDrawBracket = () => {
        drawBracket = (matchCol?.length ?? 0) % 2 === 0;
    };
    $: setDrawBracket();

    const duos = Math.floor((matchCol?.length ?? 0) / 2);
    for (let i = 0; i < duos; i++) {
        anchors[i] = {
            t: null,
            b: null,
            yTop: null,
            yBottom: null,
            yMiddle: null,
            xLeft: null,
            xMiddle: null,
            xRight: null,
        };
    }

    const getPlayoffName = (manager, bye, season) => {
        if (bye && !manager) return 'BYE';
        if (!manager) return '';
        return getTeamNameFromTeamManagers(leagueTeamManagers, manager, season);
    };

    const calculatePoints = (allPoints) => {
        let totalPoints = 0;
        for (const k in allPoints ?? {}) {
            const points = allPoints[k];
            if (!points) continue;
            for (const point of points) {
                totalPoints += point ?? 0;
            }
        }
        return round(totalPoints);
    };

    const calculatePotentialPoints = (startersWeeks, ix, p) => {
        if (!startersWeeks) return 0;
        let totalPoints = 0;
        for (const k in startersWeeks) {
            const starters = startersWeeks[k];
            if (!starters) continue;
            const i = ix + parseInt(k) - 1;
            for (const starter of starters) {
                const val = parseFloat(
                    players?.[starter]?.wi?.[playoffsStart - i]?.p ?? 0
                );
                totalPoints += isNaN(val) ? 0 : val;
            }
        }
        return round(totalPoints);
    };

    let el;
    let labelY = 0;
    let innerWidth;

    const resize = () => {
        const colTop = el?.getBoundingClientRect()?.top ?? 0;
        const colLeft = el?.getBoundingClientRect()?.left ?? 0;
        const colRight = el?.getBoundingClientRect()?.right ?? 0;
        const colWidth = colRight - colLeft;
        for (const key in anchors) {
            const tTop = anchors[key]?.t?.getBoundingClientRect()?.top ?? 0;
            const tBottom = anchors[key]?.t?.getBoundingClientRect()?.bottom ?? 0;
            if (labelY === 0 && tTop > 0) labelY = tTop - colTop - 25;

            const tLeft = anchors[key]?.t?.getBoundingClientRect()?.left ?? 0;
            const tRight = anchors[key]?.t?.getBoundingClientRect()?.right ?? 0;
            const bTop = anchors[key]?.b?.getBoundingClientRect()?.top ?? 0;
            const bBottom = anchors[key]?.b?.getBoundingClientRect()?.bottom ?? 0;

            anchors[key].yTop = (tBottom + tTop) / 2 - colTop;
            anchors[key].yBottom = (bBottom + bTop) / 2 - colTop;
            anchors[key].yMiddle = (anchors[key].yTop + anchors[key].yBottom) / 2;

            anchors[key].xLeft = (tRight + tLeft) / 2 - colWidth * ix;
            anchors[key].xMiddle = anchors[key].xLeft + colWidth / 2;
            anchors[key].xRight = anchors[key].xLeft + colWidth;
        }
    };

    $: resize(innerWidth);

    const changeSelection = (m, opponent) => {
        if (m === selected || !opponent) return;
        selected = m;
    };
</script>

<svelte:window bind:innerWidth={innerWidth} />

<div class="bracketColumn" bind:this={el}>
    {#if matchCol?.length}
        <p class="label" style="top: {labelY}px;">{label}</p>
    {/if}

    {#each matchCol ?? [] as matchups, inx}
        <div
            class="match
                {matchups?.[0]?.m === selected ? ' selected' : ''}
                {matchups?.[1]?.roster_id ? ' clickable' : ''}"
            bind:this={anchors[Math.floor(inx / 2)]?.[inx % 2 === 0 ? 't' : 'b']}
            on:click={() => changeSelection(matchups?.[0]?.m, matchups?.[1]?.roster_id)}
        >
            {#each matchups ?? [] as matchup}
                <div class="manager">
                    <div class="avatarPointsBlock">
                        {#if !matchup?.roster_id}
                            <span />
                        {/if}
                        {#if matchup?.roster_id || (!matchups?.bye && !matchup?.roster_id)}
                            <img
                                class="avatar{!matchups?.bye && !matchup?.roster_id ? ' avatarBye' : ''}"
                                src={getAvatarFromTeamManagers(leagueTeamManagers, matchup?.roster_id, leagueTeamManagers?.currentYear)}
                                alt="team avatar"
                            />
                        {/if}
                        {#if matchup?.roster_id}
                            <div class="points">
                                <div class="actualPoints">{calculatePoints(matchup?.points)}</div>
                                <div class="projectedPoints">{calculatePotentialPoints(matchup?.starters, ix, players)}</div>
                            </div>
                        {:else}
                            <span />
                        {/if}
                    </div>
                    <div class="name{matchups?.bye && !matchup?.roster_id ? ' bye' : ''}">
                        {getPlayoffName(matchup?.roster_id, matchups?.bye, leagueTeamManagers?.currentYear)}
                    </div>
                </div>
            {/each}
        </div>

        {#if drawBracket && inx % 2 === 0}
            <svg class="lineParent">
                <line
                    stroke-width="2px"
                    stroke="#ccc"
                    x1="{anchors[Math.floor(inx / 2)]?.xLeft}"
                    y1="{anchors[Math.floor(inx / 2)]?.yTop}"
                    x2="{anchors[Math.floor(inx / 2)]?.xMiddle}"
                    y2="{anchors[Math.floor(inx / 2)]?.yTop}"
                    class="line"
                />
                <line
                    stroke-width="2px"
                    stroke="#ccc"
                    x1="{anchors[Math.floor(inx / 2)]?.xMiddle}"
                    y1="{anchors[Math.floor(inx / 2)]?.yTop}"
                    x2="{anchors[Math.floor(inx / 2)]?.xMiddle}"
                    y2="{anchors[Math.floor(inx / 2)]?.yBottom}"
                    class="line"
                />
                <line
                    stroke-width="2px"
                    stroke="#ccc"
                    x1="{anchors[Math.floor(inx / 2)]?.xMiddle}"
                    y1="{anchors[Math.floor(inx / 2)]?.yMiddle}"
                    x2="{anchors[Math.floor(inx / 2)]?.xRight}"
                    y2="{anchors[Math.floor(inx / 2)]?.yMiddle}"
                    class="line"
                />
                <line
                    stroke-width="2px"
                    stroke="#ccc"
                    x1="{anchors[Math.floor(inx / 2)]?.xLeft}"
                    y1="{anchors[Math.floor(inx / 2)]?.yBottom}"
                    x2="{anchors[Math.floor(inx / 2)]?.xMiddle}"
                    y2="{anchors[Math.floor(inx / 2)]?.yBottom}"
                    class="line"
                />
            </svg>
        {/if}

        {:else}
            <div class="match spacer" />
    {/each}
</div>
