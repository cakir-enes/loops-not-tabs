<script>
	import { onMount, afterUpdate } from "svelte";
	import Loops from "./Loops.svelte";
	import Mousetrap from "mousetrap";

	let player;
	let loops = [];
	let isRecording = false;
	let loop = { start: null, end: null };
	let loadYT;
	let activeLoop;
	let time;
	var tag = document.createElement("script");

	tag.src = "https://www.youtube.com/iframe_api";
	var firstScriptTag = document.getElementsByTagName("script")[0];
	firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

	const playVideo = () => player && player.playVideo();
	const pauseVideo = () => player && player.pauseVideo();
	const currentState = () => player && player.getPlayerState();
	const currentTime = () => player && player.getCurrentTime();
	const stopLoop = () => activeLoop && clearInterval(activeLoop);

	function handleLoop() {
	  console.log(isRecording);
	  if (isRecording) {
	    loop.end = currentTime();
	    loops = [...loops, loop];
	    loop = { start: null, end: null };
	    isRecording = false;
	  } else {
	    loop.start = currentTime();
	    isRecording = true;
	  }
	}

	function startLoop({ start, end }) {
	  stopLoop();
	  console.log("Started loop");
	  activeLoop = setInterval(() => {
	    let diff = currentTime() - end;
	    if (diff > 0) {
	      console.log("Loop back " + diff);
	      player.seekTo(start);
	    }
	  }, 500);
	}


	onMount(() => {
	  if (!loadYT) {
	    loadYT = new Promise(resolve => {
	      const tag = document.createElement("script");
	      tag.src = "https://www.youtube.com/iframe_api";
	      const firstScriptTag = document.getElementsByTagName("script")[0];
	      firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
	      window.onYouTubeIframeAPIReady = () => resolve(window.YT);
	    });
	  }
	  loadYT.then(YT => {
	    player = new YT.Player("player", {
	      height: 300,
	      width: 400,
	      videoId: "M7lc1UVf-VE"
	    });
	  });
		
		  Mousetrap.bind("space", function() {
	    if ((currentState() == YT.PlayerState.PLAYING)) {
	      player.pauseVideo();
	    } else {
	      player.playVideo();
	    }
	    return false;
		})
		Mousetrap.bind("r", function() {
			handleLoop()
		}) 
	});
	const btnStyle = "f6 link dim br-pill ph3 pv2 mb2 dib white bg-black";
</script>b

<div>
		<div style="background-color:red"> 
			<h1>ASDDSA</h1>	
		</div>
</div>
<!-- <div class="mw5 mw6-ns center pt4" style="background-color:gray"> -->
  <div class="vh-75 cover bg-center" style="background-color:black">
		<div style="height:100%;width:100%" sandbox="allow-scripts allow-same-origin allow-presentation" id='player' />
  </div>
	<div class="flex items-center justify-center pa4">
		<button class={btnStyle} on:click={handleLoop}>{isRecording ? "Save A/B" : "STOP A/B"}</button>
		<button class={btnStyle} on:click={playVideo}>Play</button>
		<button class={btnStyle} on:click={() => player.seekTo(loop.start)}>{`Go@${loop.start}/${loop.end}`}</button>
		<button class={btnStyle} on:click={stopLoop}>STOP LOOP</button>
	</div>
<!-- </div> -->
	<Loops loops={loops} startLoop={startLoop} />

<style>
</style>