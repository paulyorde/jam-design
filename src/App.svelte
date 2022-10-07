<script>
  import { fade } from 'svelte/transition';
  import "chota";
  import {
    Row,
    Col,
    Container
  } from "svelte-chota";
  import * as Tone from "tone";
  import { saveAs } from "file-saver";
  import * as encoder from "audio-encoder";

  let toneRecorder = null;
  let toneMic;
  let toneContext;
  let toneRecordBlob = null;
  let player;
  let atcStream;
  let reverb = null;
  let delay = null;
  let chorus = null;
  let dirt = null;
  let active = true
  let activeCtrls = true
  let activeRadio = false
  let recordStatus = {
    isRecording: false,
    record: "https://icongr.am/jam/mic-f.svg?size=45&color=f5f0f0",
    stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0",
  };
  let recoredSrc = recordStatus.record;
  let playStatus = {
    isPlaying: false,
    play: "https://icongr.am/clarity/play.svg?size=45&color=f5f0f0",
    stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0",
  };
  let powerStatus = {
    micOn: false,
    reverbOn: false,
    delayOn: false,
    chorusOn: false,
    distortionOn: false,
    reverbImgSrcOff: "https://icongr.am/jam/power.svg?size=30&color=black",
    reverbImgSrcOn: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0",
  
  };
  let playSrc = playStatus.play;
  let reverbImgSrc = powerStatus.reverbImgSrcOn;
  let delayImgSrc = "https://icongr.am/jam/power.svg?size=30&color=f5f0f0"
  let chorusImgSrc = "https://icongr.am/jam/power.svg?size=30&color=f5f0f0"
  let distortionImgSrc = "https://icongr.am/jam/power.svg?size=30&color=f5f0f0"

  
    /**
     * listener $effect
     * example recorder
     * would enable mic to be muted on delay when mic status changes w/o effects knowledge
     * could disconnect effects on record stop
    */
  
  //  $: if(powerStatus.reverbOn) {
  //   console.log('reverb listener start')
  //   reverbImgSrc = powerStatus.reverbImgSrcOff;
  //   /**
  //    * if toneMic = currently toneMic is not being disposed - so this will be on and will connect reverb
  //   */
  //   if(toneMic) {
  //     toneMic.connect(reverb);
  //     console.log('reverb listener: on should be true', powerStatus.reverbOn)
  //   }

  //  } else {
  //   reverbImgSrc = powerStatus.reverbImgSrcOn

  //   if(toneMic && reverb) {
  //     toneMic.disconnect(reverb)
  //     reverb.dispose()
  //     reverb = null
  //     console.log('reverb listener on should be false', powerStatus.reverbOn)
  //   }
  //  }

  async function getMediaDevice() {
    activeCtrls = !activeCtrls
    activeRadio = !activeRadio

    /**
     * start/resume the Audio Context state = 'running'
    */
    await Tone.start();

    if (!toneMic) {
      toneMic = new Tone.UserMedia().toDestination();
      toneContext = Tone.context;

      toneMic.open().then((stream) => {
        console.log("mic", stream);
        atcStream = stream;
      });
    }
  }

  async function startRecord() {
    // console.log("start record stream", atcStream);

    if(!toneRecorder) {
      toneRecorder = new Tone.Recorder();
      toneMic.connect(toneRecorder);

    }
    /**
     * Recorder state = stopped on start and stop even on 1st time
    */
    console.log('recorder started', toneRecorder)
    /**
     * check if context state = 'running/stopped'
    */
    // await Tone.context.resume();
    

    if(reverb) {
      reverb.connect(toneRecorder)
      console.log('recording w/ reverb', reverb)
    }
    if(delay) {
      delay.connect(toneRecorder);
      console.log('recording w/ delay', delay)
    }
    if(chorus) {
      chorus.connect(toneRecorder);
      console.log('recording w/ chrs', chorus)
    }
    if(dirt) {
      dirt.connect(toneRecorder);
      console.log('recording w/ dirt', dirt)
    }

    await toneRecorder.start();
  }

  async function stopRecord() {
    // if(toneRecordBlob) {
    //   toneRecordBlob = "";
    // }
    console.log('recorder stopped', toneRecorder)
    toneRecordBlob = await toneRecorder.stop();
    // console.log('blob:', toneRecordBlob)
    // console.log("stop record stream ", atcStream);
    // console.log("stop record tonecontext::", toneContext);
    /**
     * these cause not playing back 1st recording after starting record 2nd time
     * currently will play 1st recording, after 2nd recording started and stopped
     */
    // await toneMic.disconnect(toneRecorder);
    // toneRecorder.dispose()
    Tone.context.dispose();
    toneContext.rawContext.suspend();
   
    /**
     * close cause effects to not be available
    */
    // toneMic.close();
  }

  async function play() {
    
    toneRecordBlob
      .arrayBuffer()
      // decode is expensive - does tone make it faster - can this passed to worker
      .then((arrayBuffer) => toneContext.decodeAudioData(arrayBuffer))
      .then(async (audioBuffer) => {
        console.log("blob", audioBuffer);
        await Tone.context.resume();

        /** 
         * todo:
         * only needs one player
         * needs the new url of next recording
        */
        // if(!player) {
          player = new Tone.Player({ url: audioBuffer }).toDestination();
        // }
        player.start();
      });
  }

  function stop() {
    player.stop();
    console.log("stop");
    // toneContext.rawContext.suspend();
  }

  /**
   * V2
   * download to computer
   * OR
   * dowload to track pad (vrs, chrs, bridge, etc)
   */
  async function download() {
    console.log("blob");
    toneRecordBlob
      .arrayBuffer()
      .then((arrayBuffer) => toneContext.decodeAudioData(arrayBuffer))
      .then((audioBuffer) => {
        console.log("blob", audioBuffer);

        // @ts-ignore
        encoder(
          audioBuffer,
          "WAV",
          (v) => console.log("happeing now", v),
          (blob) => {
            saveAs(blob, "sound.mp3");
          }
        );
      });
  }

  function toggleRecordStatus() {
    recordStatus.isRecording = !recordStatus.isRecording;
    if (recordStatus.isRecording) {
      recoredSrc = recordStatus.stop;
      startRecord();
      toneMic.mute = false;
    } else {
      recoredSrc = recordStatus.record;
      stopRecord();
      toneMic.mute = true;
    }
  }

  function togglePlayStatus() {
    playStatus.isPlaying = !playStatus.isPlaying;

    if (playStatus.isPlaying) {
      playSrc = playStatus.stop;
      play();
    } else {
      playSrc = playStatus.play;
      stop();
    }
  }

  /**
   * todo
   * start effects first , then start record.
   * 
   * in recorder listener: if(reverb) reverb.connect(toneRecorder) else // reverb.disconnect(toneRecorder);
   */
  async function toggleReverb() {
    if (powerStatus.reverbOn) {
      powerStatus.reverbOn = false;
      // reverb = null
      // console.log("reverb node", reverb);
      console.log("reverb status on should be false", powerStatus.reverbOn);
    } else {
      powerStatus.reverbOn = true;
      // console.log("reverb node", reverb);
      console.log("reverb status on should be true", powerStatus.reverbOn);
    }

    if(toneMic.mute) {
        toneMic.mute = false
      }
    if (!reverb || reverb['_wasDisposed'] === true) {
      console.log('start reverb')
      // await Tone.start();
      
      // await Tone.context.resume()
      reverb = new Tone.Reverb({
        wet: 1,
        decay: .9,
        preDelay: .4,
      }).toDestination();

      toneMic.connect(reverb);
      console.log('reverb listener on should be true', powerStatus.reverbOn)
      
    } else {
      toneMic.disconnect(reverb)
      reverb.dispose()
      reverb = null
      console.log('reverb listener on should be false', powerStatus.reverbOn)
    }

   
  }

  function toggleDelay() {
    if(powerStatus.delayOn) {
      powerStatus.delayOn = false
    } else {
      powerStatus.delayOn = true
    }

    if(toneMic.mute) {
      toneMic.mute = false;
    }
    
    // needs tone.start
    if(!delay || delay['_wasDisposed'] === true) {
      const options = { debug: true, delayTime: "4n", feedBack: 0.4 };
      delay = new Tone.PingPongDelay(options).toDestination();
      toneMic.connect(delay);
      console.log("delay", delay, "mic is muted: ", toneMic.mute);
    } else {
      delay.disconnect()
      delay.dispose()
      delay = null;
      console.log("delay", delay, "mic is muted: ", toneMic.mute);
    }

    console.log("delay after", delay, "mic is muted: ", toneMic.mute);
    
  }

  function toggleChours() {
    if(powerStatus.chorusOn) {
      powerStatus.chorusOn = false
    } else {
      powerStatus.chorusOn = true
    }

    if(toneMic.mute) {
      toneMic.mute = false;
    }

    if(!chorus || chorus['_wasDisposed'] === true) {
      chorus = new Tone.Chorus(2, .1, 1).toDestination();
      toneMic.connect(chorus);
      console.log("chours", chorus);
    } else {
      chorus.disconnect()
      chorus.dispose()
      chorus = null;
      console.log("chours", chorus);
    }
  }

  /**
   * https://tonejs.github.io/docs/14.7.77/Distortion
   * todo: research options overtone , input:gain, wet/dry to fix latency
   */
  function toggleDistortion() {
    if(powerStatus.distortionOn) {
      powerStatus.distortionOn = false
    } else {
      powerStatus.distortionOn = true
    }

    if(toneMic.mute) {
      toneMic.mute = false;
    }

    if(!dirt || dirt['_wasDisposed'] === true) {
      dirt = new Tone.Distortion(1).toDestination();
      toneMic.connect(dirt);
      console.log('distortion', dirt)
    } else {
      dirt.disconnect()
      dirt.dispose()
      dirt = null;
      console.log("chours", dirt);
    }
  }
</script>


<main>
  <!-- Top Bar -->
  <Container>
    <Row style="color: #5b87c2; font-size: x-large; height:200px; font-family: fantasy;margin-top:10px;">
      <Col size="6">
        <!-- Logo -->
        <div
          style="display: flex;
          align-items: center;"
        >
          <h6>Song Pad</h6>
          <!-- svelte-ignore a11y-missing-attribute -->
          <!-- https://icongr.am/jam/station.svg?size=26&color=acc0d3 -->
          <img src="public\icons8-radio-tower-48.png" />
        </div>
      </Col>
      <!-- Start Mic -->
      {#if !activeRadio}
      <Container>
        <Row style="margin-bottom: 20px">
          <Col size="5"></Col>
          <button on:click={getMediaDevice} class:activeRadio transition:fade>
            <!-- svelte-ignore a11y-missing-attribute -->
            <img src="public\icons8-radio-tower-48.png" />
          </button>
        </Row>
      </Container>
      {/if}
    </Row>
  </Container>

  <!-- Main Controls -->
  {#if !activeCtrls}
  <Container>
    <Row style="margin-bottom:10px">
      <Col size="5"></Col>
      <div class:activeCtrls transition:fade>
        <div class="app-ctrls--main">
          <!-- Record -->
          <!-- svelte-ignore a11y-missing-attribute -->
          <button on:click={toggleRecordStatus}>
            <img src={recoredSrc} />
          </button>
  
          <!-- Play -->
          <!-- svelte-ignore a11y-missing-attribute -->
          <button on:click={togglePlayStatus}>
            <img src={playSrc} />
          </button>
        </div>
      </div>
    </Row>

    <Row>
      <Col size="5"></Col>
      <div class:activeCtrls>
        <div class="app-ctrls--main">
          <!-- Open Effects -->
          <!-- svelte-ignore a11y-missing-attribute -->
          <button on:click={(_) => (active = !active)}>
            <img
              src="https://icongr.am/entypo/sound-mix.svg?size=45&color=f5f0f0"
            />
          </button>
          <!-- Save -->
          <!-- svelte-ignore a11y-missing-attribute -->
          <button on:click={download}>
            <img
              src="https://icongr.am/feather/download-cloud.svg?size=45&color=f5f0f0"
            />
          </button>
        </div>
      </div>
    </Row>
  </Container>
  {/if}

  <!-- Effects -->
  {#if !active}
  <Container>
    <div class:active style="margin-top: 10px;" transition:fade>
      <Row style="margin-bottom: 10px;">
        <Col size="5"></Col>
        <!-- Reverb Delay-->
        <div class="app-ctrls--main">
          <!-- svelte-ignore a11y-missing-attribute -->
          <button on:click={toggleReverb}>
            <img src={reverbImgSrc} title="Turn Reverb On/Off" />
            <h6 class="modal-ctrls--effects">Reverb</h6>
          </button>

          <button on:click={toggleDelay}>
            <!-- svelte-ignore a11y-missing-attribute -->
            <img src={delayImgSrc} />
            <h6 class="modal-ctrls--effects">Delay</h6>
          </button>
        </div>
      </Row>

      <Row>
        <Col size="5"></Col>
        <!-- Chorus Distortion -->
        <div class="app-ctrls--main">
          <button on:click={toggleChours}>
            <!-- svelte-ignore a11y-missing-attribute -->
            <img src={chorusImgSrc} />
            <h6 class="modal-ctrls--effects">Chorus</h6>
          </button>

          <button on:click={toggleDistortion}>
            <!-- svelte-ignore a11y-missing-attribute -->
            <img src={distortionImgSrc} />
            <h6 class="modal-ctrls--effects">Dirt</h6>
          </button>
        </div>
      </Row>
    </div>
  </Container>
  {/if}

</main>


<style>
  .active {
    display: none;
  }

  .activeRadio {
    display: none;
  }
  .button,
  input[type="button"],
  input[type="reset"],
  input[type="submit"],
  button {
    background-color: #5a86c1 !important;
    border: none;
    min-width: 95px;
    max-width: 95px;
  }
  .app-ctrls--main {
    padding-right: 10px;
  }

  .activeCtrls {
    display: none;
  }
  .modal-ctrls--effects {
    color: #f5f0f0;
  }

  /* input[type='range'] {
      overflow: hidden;
      width: 80px;
      -webkit-appearance: none;
      background-color: #5a86c1;
    }
    
    input[type='range']::-webkit-slider-runnable-track {
      height: 10px;
      -webkit-appearance: none !important;
      color: #5a86c1;
      margin-top: -1px;
    }
    
    input[type='range']::-webkit-slider-thumb {
      width: 10px;
      -webkit-appearance: none;
      height: 10px;
      cursor: ew-resize;
      background: #5a86c1;
      box-shadow: -80px 0 0 80px #43e5f7;
    } */

  :global(:root) {
    --bg-color: #ffffff;
    --bg-secondary-color: #5a86c1;
    --color-primary: #5a86c1;
    --color-lightGrey: #5a86c1;
    --color-grey: #5a86c1;
    --color-darkGrey: #5a86c1;
    --color-error: #5a86c1;
    --color-success: #5a86c1;
    --grid-maxWidth: 120rem;
    --grid-gutter: 2rem;
    --font-size: 1.6rem;
    --font-color: #333333;
    --font-family-sans: sans-serif;
    --font-family-mono: monaco, "Consolas", "Lucida Console", monospace;
  }

  /* theme colors
  467eaf  #1a9ce2  #35bdec  #50e5fe  #199be2 */

</style>
