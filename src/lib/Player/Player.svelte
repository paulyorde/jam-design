<script>
  let player
  let playStatus = { 
    isPlaying: false,
    play: "https://icongr.am/clarity/play.svg?size=45&color=f5f0f0",
    stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0"
  }
  let playSrc =  playStatus.play 

  async function play() {
    toneRecordBlob
      .arrayBuffer()
      .then((arrayBuffer) => toneContext.decodeAudioData(arrayBuffer))
      .then(async (audioBuffer) => {
        console.log("blob", audioBuffer);
        await Tone.context.resume();

        if(!player) {
          player = new Tone.Player({ url: audioBuffer }).toDestination();
        }
        player.start();
      });
  }

  function stop() {
    player.stop();
    console.log("stop");
    toneContext.rawContext.suspend();
  }

  function togglePlayStatus() {
    playStatus.isPlaying = !playStatus.isPlaying

    if (playStatus.isPlaying) {
      playSrc = playStatus.stop
      play()
    } else {
      playSrc = playStatus.play
      stop()
    }
  }
</script>

<!-- html -->
<button  on:click={togglePlayStatus}>
    <!-- svelte-ignore a11y-missing-attribute -->
    <img src={playSrc}  />
</button>


