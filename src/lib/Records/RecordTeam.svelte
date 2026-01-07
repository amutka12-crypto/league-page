<script>
	import {
		getAvatarFromTeamManagers,
		getTeamNameFromTeamManagers,
		renderManagerNames
	} from "$lib/utils/helperFunctions/universalFunctions";

	export let leagueTeamManagers;
	export let managerID = null;
	export let rosterID = null;
	export let year;
	export let compressed = false;
	export let points = null;

	let user = null;

	$: {
		if (managerID && leagueTeamManagers?.users) {
			user = leagueTeamManagers.users[managerID] ?? null;
		} else {
			user = null;
		}
	}
</script>
