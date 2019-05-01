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
	let re = /(?:youtube(?:-nocookie)?\.com\/(?:[^\/\n\s]+\/\S+\/|(?:v|e(?:mbed)?)\/|\S*?[?&]v=)|youtu\.be\/)([a-zA-Z0-9_-]{11})/
	let videoUrl = "";
	var tag = document.createElement("script");
	let myConfObj = {
			iframeMouseOver : false
		}


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
		player.seekTo(start)
	  activeLoop = setInterval(() => {
	    let diff = currentTime() - end;
	    if (diff > 0) {
	      player.seekTo(start);
	    }
	  }, 450);
	}

	function stopLoop() {
	  isPlayingLoop = false;
		clearInterval(activeLoop);
		activeLoop = null
	}


	function loadNewVideo() {
		loops = []
		let id = videoUrl.match(re)[1]
		if (id) {
			player.loadVideoById(id)
			videoUrl = ""
		} else {
			videoUrl = "Ooops"
		}
	}

	onMount(() => {
	  if (!loadYT) {
	    loadYT = new Promise(resolve => {
	      const tag = document.createElement("script");
	      tag.src = "https://youtube.com/iframe_api";
	      const firstScriptTag = document.getElementsByTagName("script")[0];
	      firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
	      window.onYouTubeIframeAPIReady = () => resolve(window.YT);
	    });
	  }

	  loadYT.then(YT => {
	    player = new YT.Player("player", {
	      height: "100%",
				width: "100%",
				videoId: "A8CoUmmOKpI",
				playerVars: {"rel": 0},
	      events: {
	        onStateChange: e => (isPlaying = e.data == YT.PlayerState.PLAYING)
	      }
			});
			window.addEventListener('blur',function(){
			if(myConfObj.iframeMouseOver){
			}
		});

		document.getElementById('player').addEventListener('mouseover',function(){
			myConfObj.iframeMouseOver = true;
		});
		document.getElementById('player').addEventListener('mouseout',function(){
				myConfObj.iframeMouseOver = false;
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

</script>


<div class="ui small center aligned segment">
	<div class="ui fluid action input">
			<input type="text" bind:value={videoUrl} placeholder="Paste video url...">
			<button class="ui button" on:click={loadNewVideo}>Load</button>
		</div>
</div>

<div class="flex-wrapper">
	<div class="video-wrapper">
		<div class="ratio-keeper">
			<div class="video-frame mousetrap" sandbox="allow-scripts allow-same-origin allow-presentation" id='player' />
		</div>
	</div>
</div>

<div class="ui fluid large buttons">
		<div class="ui labeled button" tabindex="0">
				<div class={`ui red ${isPlaying ? "basic" : " "} button`} on:click={() => isPlaying ? pauseVideo() : playVideo()}>
						<i class={isPlaying ? "pause icon" : "play icon"} />{isPlaying ? "PAUSE" : "PLAY"}
				</div>
				<a class="ui basic black left pointing label">
					SPC
				</a>
		</div>

		<div class="ui labeled right floated button" tabindex="0">
				<div class="{`ui ${isRecording ? "green" : "blue"} basic button`}" on:click={handleLoop}>
						<i class="{`infinity ${isRecording ? "loading" : ""} icon`}" /> {`${isRecording ? "SAVE LOOP" : "REC LOOP"}`}
				</div>
				<a class="ui basic black left pointing label">
					R
				</a>
		</div>

			<div class="ui labeled right floated button" tabindex="0">
					<div class={`ui ${activeLoop ? "" : "basic"} black button`} on:click={stopLoop}>
							<i class="minus icon" /> KILL LOOP
					</div>
					<a class="ui basic black left pointing label">
						<i class="angle up icon" />SPC
					</a>
				</div>
</div>


<div class="ui segment">
	{#if loops.length > 0}
		<Loops loops={loops} startLoop={startLoop} />
	{:else}
		<div class="ui center aligned container">RECORD SOME LOOPS</div>
	{/if}
</div>
<style>
.flex-wrapper {
    display: flex;
		justify-content: center;
    flex-flow: row wrap;
}

.video-wrapper {
    min-width: 40%;
    max-width: 75%;
    height: 100%;
    margin:  10px 10px;
    flex: 1 1 auto;
}

@media only screen and (max-width: 500px) {
  .video-wrapper {
    min-width: 360px;
  }
}

.ratio-keeper {
    position: relative;
    padding-top: 56.25%;
}

.video-frame {
    position: absolute;
    top: 0; left: 0;
    width: 100%;
    height: 100%;
}
</style>