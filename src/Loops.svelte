<script>
  import Mousetrap from "mousetrap";
  
  export let loops = [];
  export let startLoop;

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


  $: {
	  loops.forEach((loop, i) => Mousetrap.bind(`${i}`, () => startLoop(loop)))
  }
</script>

<div class="ma2">
  {#each loops as loop, i}
  <!-- {Mousetrap.bind(`${i}`, () => startLoop(loop))} -->
  <button class="ma1" on:click={() => startLoop(loop)}>{`${i}|${formatTime(loop.start)}-${formatTime(loop.end)}`}</button>
	<!-- {createButton(loop, i)} -->
  {/each}
</div>

