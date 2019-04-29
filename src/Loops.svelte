<script>
export let loops = []
export let startLoop

function formatTime(sec) {
		  let sec_num = parseInt(sec, 10);
		  let hours = Math.floor(sec_num / 3600);
		  let minutes = Math.floor((sec_num - hours * 3600) / 60);
		  let seconds = sec_num - hours * 3600 - minutes * 60;
		  if (hours < 10) hours = "0" + hours;
		  if (minutes < 10) minutes = "0" + minutes;
		  if (seconds < 10) seconds = "0" + seconds;
		  return (hours > 0 ? hours + ":" : "") + minutes + ":" + seconds;
		}

</script>

<div class="buttons">
	{#each loops as loop}
	<button class="button" on:click={() => startLoop(loop)}>{`${formatTime(loop.start)}-${formatTime(loop.end)}`}</button>
	{/each}
</div>

