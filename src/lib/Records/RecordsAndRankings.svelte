<script>
	import Button, { Group, Label } from '@smui/button';
	import { generateGraph, gotoManager, round } from '$lib/utils/helper';
	import DataTable, { Head, Body, Row, Cell } from '@smui/data-table';
	import RecordTeam from './RecordTeam.svelte';
	import BarChart from '$lib/BarChart.svelte';

	export let key;
	export let tradesData = [];
	export let waiversData = [];
	export let weekRecords = [];
	export let weekLows = [];
	export let seasonLongRecords = [];
	export let seasonLongLows = [];
	export let showTies = false;
	export let winPercentages = [];
	export let fptsHistories = [];
	export let lineupIQs = [];
	export let prefix;
	export let blowouts = [];
	export let closestMatchups = [];
	export let allTime = false;
	export let leagueTeamManagers = {};

	let graphs = [];
	let curTable = 0;
	let curGraph = 0;
	let iqOffset = 0;
	let tables = ["Win Percentages", "Points"];
	const year = allTime ? null : prefix;

	const changeTable = (newGraph) => {
		switch (newGraph) {
			case 0 - iqOffset:
			case 4 + (99 * iqOffset):
				curTable = 0;
				break;
			case 1 - iqOffset:
			case 2 - iqOffset:
				curTable = 1 - iqOffset;
				break;
			case 3 - iqOffset:
				curTable = 2 - iqOffset;
				break;
			case 5 - (2 * iqOffset):
			case 6 - (2 * iqOffset):
				curTable = 3 - iqOffset;
				break;
			default:
				curTable = 0;
		}
	};

	const changeGraph = (newTable) => {
		switch (newTable) {
			case 0 - iqOffset:
				if (curGraph !== 0 && curGraph !== 4) curGraph = 0;
				break;
			case 1 - iqOffset:
				if (curGraph !== 1 - iqOffset && curGraph !== 2 - iqOffset) curGraph = 1 - iqOffset;
				break;
			case 2 - iqOffset:
				curGraph = 3 - iqOffset;
				break;
			case 3 - iqOffset:
				if (curGraph !== 5 - (2 * iqOffset) && curGraph !== 6 - (2 * iqOffset)) curGraph = 5 - (2 * iqOffset);
				break;
			default:
				curGraph = 0;
		}
	};

	const setGraphs = (wD = []) => {
		const gs = [];

		if (lineupIQs?.length && lineupIQs[0]?.potentialPoints) {
			gs.push(generateGraph({
				stats: lineupIQs,
				x: "Lineup IQ",
				stat: "%",
				header: "Manager Lineup IQ",
				field: "iq",
				short: "Lineup IQ"
			}, year));

			gs.push(generateGraph({
				stats: lineupIQs,
				x: "Points",
				stat: "",
				header: "Potential Points vs Points",
				field: "potentialPoints",
				secondField: "fpts",
				short: "Potential Points"
			}, year, 10, 0));
		}

		if (winPercentages?.length) {
			gs.push(generateGraph({
				stats: winPercentages,
				x: "Wins",
				stat: "",
				header: "Team Wins",
				field: "wins",
				short: "Wins"
			}, year));

			gs.push(generateGraph({
				stats: winPercentages,
				x: "Win Percentage",
				stat: "%",
				header: "Team Win Percentages",
				field: "percentage",
				short: "Win Percentage"
			}, year));
		}

		if (fptsHistories?.length) {
			gs.push(generateGraph({
				stats: fptsHistories,
				x: "Fantasy Points",
				stat: "",
				header: "Team Fantasy Points",
				field: "fptsFor",
				short: "Fantasy Points"
			}, year));
		}

		if (key === "regularSeasonData") {
			if (tradesData?.length) gs.push(generateGraph({ stats: tradesData, x: "# of trades", stat: "", header: "Number of Trades Managers Have Made", field: "trades", short: "Trades" }, year));
			if (wD?.length) gs.push(generateGraph({ stats: wD, x: "# of Waiver Moves", stat: "", header: "Waivers Moves Managers Have Made", field: "waivers", short: "Waivers" }, year));
		}

		curGraph = 0;
		graphs = gs;
	};

	const setTransactionsAndGraphs = (wD = []) => {
		const transactions = [];
		if (!wD?.length) return transactions;

		if (wD[0]?.rosterID) {
			for (let i = 1; i <= wD.length; i++) {
				if (!tradesData.find(t => t.rosterID === i)) tradesData.push({ rosterID: i, trades: 0 });
			}
		}

		if (wD[0]?.managerID && leagueTeamManagers?.users) {
			for (const userID in leagueTeamManagers.users) {
				if (!tradesData.find(t => t.managerID === userID)) tradesData.push({ managerID: userID, trades: 0 });
			}
		}

		for (const w of wD) {
			if (!w) continue;
			let trades = 0;
			if (w.managerID) trades = tradesData.find(t => t.managerID === w.managerID)?.trades || 0;
			else if (w.rosterID) trades = tradesData.find(t => t.rosterID === w.rosterID)?.trades || 0;

			const waivers = w.waivers || 0;

			transactions.push({
				rosterID: w.rosterID || null,
				managerID: w.managerID || null,
				trades,
				waivers
			});
		}

		setGraphs(wD);
		return transactions;
	};

	const setTables = (lIQs = []) => {
		const t = ["Win Percentages", "Points"];
		if (key === "regularSeasonData") t.push("Transactions");
		if (!lIQs?.length || !lIQs[0]?.potentialPoints) iqOffset = 1;
		else t.unshift("Lineup IQs");
		tables = t;
	};

	$: transactions = setTransactionsAndGraphs(waiversData);
	$: changeTable(curGraph);
	$: changeGraph(curTable);
	$: setTables(lineupIQs);

	let innerWidth;
</script>

<svelte:window bind:innerWidth={innerWidth} />

<h4>{prefix} Records</h4>

<div class="fullFlex">

	<!-- Week Records -->
	{#if weekRecords?.length}
		<DataTable class="recordTable">
			<Head>
				<Row class="rTableHeader">
					<Cell class="header headerPrimary" colspan=4>{prefix} {key === "playoffData" ? "Playoff " : ""}Single Week Scoring Records</Cell>
				</Row>
				<Row>
					<Cell class="header rank"></Cell>
					<Cell class="header">Manager</Cell>
					<Cell class="header">Week</Cell>
					<Cell class="header">Total Points</Cell>
				</Row>
			</Head>
			<Body>
				{#each weekRecords as record, ix}
					{#if record?.rosterID}
						<Row>
							<Cell class="rank">{ix + 1}</Cell>
							<Cell class="cellName" onclick={() => gotoManager({year: record.year || prefix, leagueTeamManagers, rosterID: record.rosterID})}>
								<RecordTeam {leagueTeamManagers} rosterID={record.rosterID} year={allTime ? record.year : prefix} />
							</Cell>
							<Cell>{allTime && record.year ? record.year + " " : ""}{key === "regularSeasonData" ? "Week " : ""}{record.week}</Cell>
							<Cell>{round(record.fpts || 0)}</Cell>
						</Row>
					{/if}
				{/each}
			</Body>
		</DataTable>
	{/if}

	<!-- Week Lows -->
	{#if weekLows?.length}
		<DataTable class="recordTable">
			<Head>
				<Row>
					<Cell class="header headerPrimary" colspan=4>{prefix} {key === "playoffData" ? "Playoff " : ""}Single Week Scoring Lows</Cell>
				</Row>
				<Row>
					<Cell class="header rank"></Cell>
					<Cell class="header">Manager</Cell>
					<Cell class="header">Week</Cell>
					<Cell class="header">Total Points</Cell>
				</Row>
			</Head>
			<Body>
				{#each weekLows as low, ix}
					{#if low?.rosterID}
						<Row>
							<Cell class="rank">{ix + 1}</Cell>
							<Cell class="cellName" onclick={() => gotoManager({year: low.year || prefix, leagueTeamManagers, rosterID: low.rosterID})}>
								<RecordTeam {leagueTeamManagers} rosterID={low.rosterID} year={allTime ? low.year : prefix} />
							</Cell>
							<Cell>{allTime && low.year ? low.year + " " : ""}{key === "regularSeasonData" ? "Week " : ""}{low.week}</Cell>
							<Cell>{round(low.fpts || 0)}</Cell>
						</Row>
					{/if}
				{/each}
			</Body>
		</DataTable>
	{/if}

	<!-- Season Long Records -->
	{#if allTime && key === "regularSeasonData" && seasonLongRecords?.length}
		<DataTable class="recordTable">
			<Head>
				<Row>
					<Cell class="header headerPrimary" colspan=5>All-Time Highest Season Points<span class="italic">Ranked by PPG</span></Cell>
				</Row>
				<Row>
					<Cell class="header rank"></Cell>
					<Cell class="header">Manager</Cell>
					<Cell class="header">Year</Cell>
					<Cell class="header">Total Points</Cell>
					<Cell class="header">PPG</Cell>
				</Row>
			</Head>
			<Body>
				{#each seasonLongRecords as record, ix}
					{#if record?.rosterID}
						<Row>
							<Cell class="rank">{ix + 1}</Cell>
							<Cell class="cellName" onclick={() => gotoManager({year: record.year, leagueTeamManagers, rosterID: record.rosterID})}>
								<RecordTeam {leagueTeamManagers} rosterID={record.rosterID} year={record.year} />
							</Cell>
							<Cell>{record.year}</Cell>
							<Cell>{round(record.fpts || 0)}</Cell>
							<Cell>{record.fptsPerGame || 0}</Cell>
						</Row>
					{/if}
				{/each}
			</Body>
		</DataTable>
	{/if}

	<!-- Season Long Lows -->
	{#if allTime && key === "regularSeasonData" && seasonLongLows?.length}
		<DataTable class="recordTable">
			<Head>
				<Row>
					<Cell class="header headerPrimary" colspan=5>All-Time Lowest Season Points<span class="italic">Ranked by PPG</span></Cell>
				</Row>
				<Row>
					<Cell class="header rank"></Cell>
					<Cell class="header">Manager</Cell>
					<Cell class="header">Year</Cell>
					<Cell class="header">Total Points</Cell>
					<Cell class="header">PPG</Cell>
				</Row>
			</Head>
			<Body>
				{#each seasonLongLows as record, ix}
					{#if record?.rosterID}
						<Row>
							<Cell class="rank">{ix + 1}</Cell>
							<Cell class="cellName" onclick={() => gotoManager({year: record.year, leagueTeamManagers, rosterID: record.rosterID})}>
								<RecordTeam {leagueTeamManagers} rosterID={record.rosterID} year={record.year} />
							</Cell>
							<Cell>{record.year}</Cell>
							<Cell>{round(record.fpts || 0)}</Cell>
							<Cell>{record.fptsPerGame || 0}</Cell>
						</Row>
					{/if}
				{/each}
			</Body>
		</DataTable>
	{/if}

	<!-- Blowouts -->
	{#if blowouts?.length}
		<DataTable class="recordTable">
			<Head>
				<Row>
					<Cell class="header headerPrimary" colspan=4>{prefix} Largest {key === "playoffData" ? "Playoff " : ""}Blowouts</Cell>
				</Row>
				<Row>
					<Cell class="header rank"></Cell>
					<Cell class="header">Matchup</Cell>
					<Cell class="header">Week</Cell>
					<Cell class="header">Differential</Cell>
				</Row>
			</Head>
			<Body>
				{#each blowouts as b, ix}
					{#if b?.home?.rosterID && b?.away?.rosterID}
						<Row>
							<Cell class="rank">{ix + 1}</Cell>
							<Cell class="cellName differentialName">
								<div class="vsRecord">
									<div onclick={() => gotoManager({year: b.year || prefix, leagueTeamManagers, rosterID: b.home.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={b.home.rosterID} year={allTime ? b.year : prefix} compressed={true} points={round(b.home.fpts || 0)} />
									</div>
									<p class="vs">vs</p>
									<div onclick={() => gotoManager({year: b.year || prefix, leagueTeamManagers, rosterID: b.away.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={b.away.rosterID} year={allTime ? b.year : prefix} compressed={true} points={round(b.away.fpts || 0)} />
									</div>
								</div>
							</Cell>
							<Cell>{allTime && b.year ? b.year + " " : ""}{key === "regularSeasonData" ? "Week " : ""}{b.week}</Cell>
							<Cell>{round(b.differential || 0)}</Cell>
						</Row>
					{/if}
				{/each}
			</Body>
		</DataTable>
	{/if}

	<!-- Closest Matchups -->
	{#if closestMatchups?.length}
		<DataTable class="recordTable">
			<Head>
				<Row>
					<Cell class="header headerPrimary" colspan=4>{prefix} Narrowest {key === "playoffData" ? "Playoff " : ""}Wins</Cell>
				</Row>
				<Row>
					<Cell class="header rank"></Cell>
					<Cell class="header">Matchup</Cell>
					<Cell class="header">Week</Cell>
					<Cell class="header">Differential</Cell>
				</Row>
			</Head>
			<Body>
				{#each closestMatchups as c, ix}
					{#if c?.home?.rosterID && c?.away?.rosterID}
						<Row>
							<Cell class="rank">{ix + 1}</Cell>
							<Cell class="cellName differentialName">
								<div class="vsRecord">
									<div onclick={() => gotoManager({year: c.year || prefix, leagueTeamManagers, rosterID: c.home.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={c.home.rosterID} year={allTime ? c.year : prefix} compressed={true} points={round(c.home.fpts || 0)} />
									</div>
									<p class="vs">vs</p>
									<div onclick={() => gotoManager({year: c.year || prefix, leagueTeamManagers, rosterID: c.away.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={c.away.rosterID} year={allTime ? c.year : prefix} compressed={true} points={round(c.away.fpts || 0)} />
									</div>
								</div>
							</Cell>
							<Cell>{allTime && c.year ? c.year + " " : ""}{key === "regularSeasonData" ? "Week " : ""}{c.week}</Cell>
							<Cell>{round(c.differential || 0)}</Cell>
						</Row>
					{/if}
				{/each}
			</Body>
		</DataTable>
	{/if}

</div>

<h4>{prefix} {key === "playoffData" ? "Playoff " : ""}Rankings</h4>

{#if graphs?.length}
	<BarChart {graphs} bind:curGraph={curGraph} {leagueTeamManagers} />
{/if}

<div class="rankingHolder">
	<div class="rankingInner" style="margin-left: -{100 * curTable}%;">
		<!-- Lineup IQ Rankings -->
		{#if lineupIQs?.length && lineupIQs[0]?.potentialPoints}
			<div class="rankingTableWrapper">
				<DataTable class="rankingTable">
					<Head>
						<Row>
							<Cell class="header headerPrimary" colspan=5>
								{prefix} {key === "playoffData" ? "Playoff " : ""}Lineup IQ Rankings
								<div class="subTitle">The percentage of potential points each manager has captured</div>
							</Cell>
						</Row>
						<Row>
							<Cell class="header"></Cell>
							<Cell class="header">Manager</Cell>
							<Cell class="header">Lineup IQ</Cell>
							<Cell class="header">Points</Cell>
							<Cell class="header">Potential Points</Cell>
						</Row>
					</Head>
					<Body>
						{#each lineupIQs as l, ix}
							{#if l?.rosterID}
								<Row>
									<Cell class="rank">{ix + 1}</Cell>
									<Cell class="cellName" onclick={() => gotoManager({year: l.year || prefix, leagueTeamManagers, rosterID: l.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={l.rosterID} year={allTime ? l.year : prefix} />
									</Cell>
									<Cell>{round(l.iq || 0)}%</Cell>
									<Cell>{round(l.fpts || 0)}</Cell>
									<Cell>{round(l.potentialPoints || 0)}</Cell>
								</Row>
							{/if}
						{/each}
					</Body>
				</DataTable>
			</div>
		{/if}

		<!-- Win Percentages -->
		{#if winPercentages?.length}
			<div class="rankingTableWrapper">
				<DataTable class="rankingTable">
					<Head>
						<Row>
							<Cell class="header headerPrimary" colspan=4>
								{prefix} {key === "playoffData" ? "Playoff " : ""}Win Percentage Rankings
							</Cell>
						</Row>
						<Row>
							<Cell class="header"></Cell>
							<Cell class="header">Manager</Cell>
							<Cell class="header">Wins</Cell>
							<Cell class="header">Win %</Cell>
						</Row>
					</Head>
					<Body>
						{#each winPercentages as w, ix}
							{#if w?.rosterID}
								<Row>
									<Cell class="rank">{ix + 1}</Cell>
									<Cell class="cellName" onclick={() => gotoManager({year: w.year || prefix, leagueTeamManagers, rosterID: w.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={w.rosterID} year={allTime ? w.year : prefix} />
									</Cell>
									<Cell>{w.wins || 0}</Cell>
									<Cell>{w.percentage || 0}%</Cell>
								</Row>
							{/if}
						{/each}
					</Body>
				</DataTable>
			</div>
		{/if}

		<!-- FPTS Histories -->
		{#if fptsHistories?.length}
			<div class="rankingTableWrapper">
				<DataTable class="rankingTable">
					<Head>
						<Row>
							<Cell class="header headerPrimary" colspan=3>
								{prefix} {key === "playoffData" ? "Playoff " : ""}Points Rankings
							</Cell>
						</Row>
						<Row>
							<Cell class="header"></Cell>
							<Cell class="header">Manager</Cell>
							<Cell class="header">Total Points</Cell>
						</Row>
					</Head>
					<Body>
						{#each fptsHistories as f, ix}
							{#if f?.rosterID}
								<Row>
									<Cell class="rank">{ix + 1}</Cell>
									<Cell class="cellName" onclick={() => gotoManager({year: f.year || prefix, leagueTeamManagers, rosterID: f.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={f.rosterID} year={allTime ? f.year : prefix} />
									</Cell>
									<Cell>{round(f.fptsFor || 0)}</Cell>
								</Row>
							{/if}
						{/each}
					</Body>
				</DataTable>
			</div>
		{/if}

		<!-- Transactions -->
		{#if transactions?.length}
			<div class="rankingTableWrapper">
				<DataTable class="rankingTable">
					<Head>
						<Row>
							<Cell class="header headerPrimary" colspan=4>
								{prefix} {key === "playoffData" ? "Playoff " : ""}Transactions
							</Cell>
						</Row>
						<Row>
							<Cell class="header"></Cell>
							<Cell class="header">Manager</Cell>
							<Cell class="header">Trades</Cell>
							<Cell class="header">Waivers</Cell>
						</Row>
					</Head>
					<Body>
						{#each transactions as t, ix}
							{#if t?.rosterID}
								<Row>
									<Cell class="rank">{ix + 1}</Cell>
									<Cell class="cellName" onclick={() => gotoManager({year: prefix, leagueTeamManagers, rosterID: t.rosterID})}>
										<RecordTeam {leagueTeamManagers} rosterID={t.rosterID} year={prefix} />
									</Cell>
									<Cell>{t.trades}</Cell>
									<Cell>{t.waivers}</Cell>
								</Row>
							{/if}
						{/each}
					</Body>
				</DataTable>
			</div>
		{/if}

	</div>
</div>

<div class="buttonHolder">
	<Group variant="outlined">
		{#each tables as table, ix}
			<Button class="selectionButtons" onclick={() => curTable = ix} variant={curTable === ix ? "raised" : "outlined"}>
				<Label>{table}</Label>
			</Button>
		{/each}
	</Group>
</div>
