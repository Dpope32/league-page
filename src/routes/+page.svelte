<script>
    import LinearProgress from '@smui/linear-progress';
    import { getNflState, leagueName, getAwards, getLeagueTeamManagers, homepageText, managers, gotoManager, enableBlog, waitForAll } from '$lib/utils/helper';
    import { Transactions, PowerRankings, HomePost } from '$lib/components';
    import { getAvatarFromTeamManagers, getTeamFromTeamManagers } from '$lib/utils/helperFunctions/universalFunctions';

    const nflState = getNflState();
    const podiumsData = getAwards();
    const leagueTeamManagersData = getLeagueTeamManagers();
</script>

<style>
    #home {
        display: flex;
        flex-direction: column; /* Stack children vertically */
        position: relative;
        overflow-y: hidden;
        z-index: 1;
    }

    #main {
        flex-grow: 1;
        width: 100%; /* Ensure full width */
        margin: 0 auto;
        padding: 60px 20px; /* Adjust padding as needed */
    }

    .text {
        padding: 0 20px; /* Reduced padding for better mobile view */
        max-width: 100%; /* Allow text to span full width */
        margin: 0 auto;
    }

    .leagueData {
        width: 100%; /* Ensure full width */
        min-width: auto; /* Remove fixed min-width */
        max-width: 100%; /* Allow to span full width */
        background-color: var(--ebebeb);
        border-left: none; /* Remove left border for vertical layout */
        box-shadow: none; /* Remove shadow for cleaner look */
        padding: 20px; /* Add padding for spacing */
    }

    .transactions {
        width: 100%; /* Full width for vertical layout */
        margin: 10px 0; /* Vertical spacing */
    }

    .center {
        text-align: center;
    }

    h6 {
        text-align: center;
    }

    .homeBanner {
        background-color: var(--blueOne);
        color: #fff;
        padding: 0.5em 0;
        font-weight: 500;
        font-size: 1.5em;
    }

    /* champ styling */
    #currentChamp {
        padding: 25px 0;
        background-color: var(--f3f3f3);
        box-shadow: none; /* Remove shadow for vertical layout */
        border-left: none; /* Remove border */
    }

    #champ {
        position: relative;
        width: 150px;
        height: 150px;
        margin: 0 auto;
        cursor: pointer;
    }

    .first {
        position: absolute;
        transform: translate(-50%, -50%);
        width: 80px;
        height: 80px;
        border-radius: 100%;
        border: 1px solid #ccc;
        left: 50%;
        top: 43%;
    }

    .laurel {
        position: absolute;
        transform: translate(-50%, -50%);
        width: 135px;
        height: auto;
        left: 50%;
        top: 50%;
    }

    h4 {
        text-align: center;
        font-size: 1.8em;
        margin: 10px;
        font-style: italic;
    }

    .label {
        display: table;
        text-align: center;
        line-height: 1.1em;
        font-size: 1.7em;
        margin: 6px auto 10px;
        cursor: pointer;
    }

    :global(.curOwner) {
        font-size: 0.75em;
        color: #bbb;
        font-style: italic;
    }

    @media (max-width: 950px) {
        #home {
            flex-direction: column; /* Keep vertical stack on smaller screens */
        }

        .leagueData {
            max-width: 100%;
            min-width: 100%;
            width: 100%;
            box-shadow: none; /* Remove box shadow for mobile view */
        }
    }
</style>

<div id="home">
    <div id="main">
        <div class="text">
            <!-- homepageText contains the intro text for your league, this gets edited in /src/lib/utils/leagueInfo.js -->
            {@html homepageText}
            <!-- Most recent Blog Post (if enabled) -->
            {#if enableBlog}
                <HomePost />
            {/if}
        </div>
        <PowerRankings />
    </div>

    <div class="leagueData">
        <div class="homeBanner">
            {#await nflState}
                <div class="center">Retrieving NFL state...</div>
                <LinearProgress indeterminate />
            {:then nflStateData}
                <div class="center">NFL {nflStateData.season} 
                    {#if nflStateData.season_type == 'pre'}
                        Preseason
                    {:else if nflStateData.season_type == 'post'}
                        Postseason
                    {:else}
                        Season - {nflStateData.week > 0 ? `Week ${nflStateData.week}` : "Preseason"}
                    {/if}
                </div>
            {:catch error}
                <div class="center">Something went wrong: {error.message}</div>
            {/await}
        </div>

        <div id="currentChamp">
            {#await waitForAll(podiumsData, leagueTeamManagersData)}
                <p class="center">Retrieving awards...</p>
                <LinearProgress indeterminate />
            {:then [podiums, leagueTeamManagers]}
                {#if podiums[0]}
                    <h4>{podiums[0].year} Fantasy Champ</h4>
                    <div id="champ" on:click={() => {if(managers.length) gotoManager({year: podiums[0].year, leagueTeamManagers, rosterID: parseInt(podiums[0].champion)})}} >
                        <img src="{getAvatarFromTeamManagers(leagueTeamManagers, podiums[0].champion, podiums[0].year)}" class="first" alt="champion" />
                        <img src="/laurel.png" class="laurel" alt="laurel" />
                    </div>
                    <span class="label" on:click={() => gotoManager({year: podiums[0].year, leagueTeamManagers, rosterID: parseInt(podiums[0].champion)})} >
                        {getTeamFromTeamManagers(leagueTeamManagers, podiums[0].champion, podiums[0].year).name}
                    </span>
                {:else}
                    <p class="center">No former champs.</p>
                {/if}
            {:catch error}
                <p class="center">Something went wrong: {error.message}</p>
            {/await}
        </div>

        <div class="transactions">
            <Transactions />
        </div>
    </div>
</div>
