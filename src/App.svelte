<script>
	import { onMount, afterUpdate } from "svelte";
	import Loops from "./Loops.svelte";
	import Mousetrap from "mousetrap";

	let player;
	let loops = [];
	let isRecording = false;
	let isPlaying = false;
	let isPlayingLoop = false;
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

	function handleLoop() {
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
	  isPlayingLoop = true;
	  activeLoop = setInterval(() => {
	    let diff = currentTime() - end;
	    if (diff > 0) {
	      player.seekTo(start);
	    }
	  }, 500);
	}

	function stopLoop() {
	  isPlayingLoop = false;
	  clearInterval(activeLoop);
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
	      videoId: "M7lc1UVf-VE",
	      events: {
	        onStateChange: e => (isPlaying = e.data == YT.PlayerState.PLAYING)
	      }
	    });
	  });

	  Mousetrap.bind("space", function() {
	    if (currentState() == YT.PlayerState.PLAYING) {
	      player.pauseVideo();
	    } else {
	      player.playVideo();
	    }
	    return false;
	  });
	  Mousetrap.bind("r", function() {
	    handleLoop();
	  });
	  Mousetrap.bind("shift+space", stopLoop);
	});

	const baseBtn = "white b pv2 ph3 bn br2 ma2";
	// const baseBtn = "white avenir ba b--black-10  br-50  w3-ns tc h2 h3-ns";
	const btnStyle = baseBtn;
	$: loopBtn = `${baseBtn} ${isRecording ? "bg-green" : "bg-dark-blue"}`;
	$: playBtn = `${baseBtn} ${isPlaying ? "bg-red" : "bg-gray"}`;
</script>

<h1 class="white bg-near-black avenir lh-solid">Slower</h1>	
<div class="vh-75 cover bg-center">
	<div style="height:100%;width:100%" sandbox="allow-scripts allow-same-origin allow-presentation" id='player' />
</div>
<div class="flex bg-near-white">
	<button class="{loopBtn}" on:click={handleLoop}>{`R | ${isRecording ? "SAVE LOOP" : "REC LOOP"}`}</button>
	<button class="{playBtn}" on:click={() => isPlaying ? pauseVideo() : playVideo()}>
		SPC
		<ion-icon name="{`${isPlaying ? "pause" : "play"}`}"></ion-icon>
	</button>
	<button class="{playBtn}" on:click={stopLoop}>{`SHIFT+SPC | ${isPlayingLoop ? "STOP" : "PLAY"}`}</button>
</div>
<Loops loops={loops} startLoop={startLoop} />
