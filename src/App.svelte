<script>
  import "chota";
  import {
    Modal,
    Button,
    Card,
    Row,
    Col,
    Container,
    Input,
  } from "svelte-chota";
  import * as Tone from "tone";
  import { saveAs } from "file-saver";
  import * as encoder from "audio-encoder";
    import Effects from "./lib/Effects.svelte";

  // these object porperties below could be here in main file passed into each component
  // or used as a store?
  // usermedia , context happens in main file
  // recorder in recorder, player in player, etc...

  let toneRecorder;
  let toneMic;
  let toneContext;
  // let toneStreamNode
  // let toneStreamSourceNode
  let toneRecordBlob;
  let player;
  let atcStream;
  let reverb = null;
  let active = true

  let modal_open = false;

  let recordStatus = {
    isRecording: false,
    record: "https://icongr.am/jam/mic-f.svg?size=45&color=f5f0f0",
    // record: "https://icongr.am/jam/mic-f.svg?size=45&color=f5f0f0",
    stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0",
    // src:
    // stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0"
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
  
   $: if(powerStatus.reverbOn) {
    console.log('reverb listener start')
    reverbImgSrc = powerStatus.reverbImgSrcOff;
    if(toneMic) {
      toneMic.connect(reverb);
      console.log('reverb listener: on should be true', powerStatus.reverbOn)
    }

   } else {
    reverbImgSrc = powerStatus.reverbImgSrcOn

    if(toneMic && reverb) {
      toneMic.disconnect(reverb)
      reverb.dispose()
      reverb = null
      console.log('reverb listener on should be false', powerStatus.reverbOn)
    }
   }

  //  $: if(powerStatus.micOn) {
  //   if(!toneMic) {
  //     toneMic = new Tone.UserMedia().toDestination();
  //     toneContext = Tone.context;
  //     // toneRecorder = new Tone.Recorder();

  //     toneMic.open().then((stream) => {
  //       console.log("mic listener on should be true", toneMic);
  //       atcStream = stream;
  //       });
  //     }
     
  //  } else {
  //   console.log('mic listener on should be false', powerStatus.micOn)
  //  }
    

  async function getMediaDevice() {
    await Tone.start();
   

    if (!toneMic) {
      // pass in { toneMic: userMedia, toneContext: audioContext }
      // -> returns copy with updated userMedia to calling method.
      toneMic = new Tone.UserMedia().toDestination();
      toneContext = Tone.context;
      toneRecorder = new Tone.Recorder();

      toneMic.open().then((stream) => {
        console.log("mic", stream);
        atcStream = stream;
      });
    }
    console.log("arter toneMic", toneMic);

    /**
     * stream state = started
     * toneContext/audioContext = running
     */
    // startRecord();
  }

  // pass in userMedia object
  async function startRecord() {
    console.log("start record stream", atcStream);
 
    await Tone.context.resume();
    
    toneMic.connect(toneRecorder);
    if(reverb) {
      reverb.connect(toneRecorder)
    }
    await toneRecorder.start();

    // toneStreamNode = toneContext.createMediaStreamDestination();
    // toneStreamSourceNode = toneContext.createMediaStreamSource(toneStreamNode.stream);
  }

  // pass in recorder, mic, context.
  async function stopRecord() {
    toneRecordBlob = await toneRecorder.stop();
    console.log("stop record stream ", atcStream);
    console.log("stop record tonecontext::", toneContext);
    /**
     * Cleanup - saving system resources
     */
    await toneMic.disconnect(toneRecorder);
    Tone.context.dispose();
    toneContext.rawContext.suspend();
    toneMic.close();
  }

  async function play() {
    toneRecordBlob
      .arrayBuffer()
      // decode is expensive - does tone make it faster - can this passed to worker
      .then((arrayBuffer) => toneContext.decodeAudioData(arrayBuffer))
      .then(async (audioBuffer) => {
        console.log("blob", audioBuffer);
        await Tone.context.resume();

        // not need to be new player each time.
        player = new Tone.Player({ url: audioBuffer }).toDestination();
        player.start();
      });
  }

  function stop() {
    player.stop();
    console.log("stop");
    toneContext.rawContext.suspend();
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
    if (!reverb || reverb['_wasDisposed'] === true) {
      console.log('start reverb')

      await Tone.start();
      await Tone.context.resume()
      // powerStatus.micOn = true
      reverb = new Tone.Reverb({
        wet: 1,
        decay: .9,
        preDelay: .4,
      }).toDestination();
    }

    if (powerStatus.reverbOn) {
      powerStatus.reverbOn = false;
      console.log("reverb node", reverb);
      console.log("reverb status on should be false", powerStatus.reverbOn);
    } else {
      powerStatus.reverbOn = true;
      console.log("reverb node", reverb);
      console.log("reverb status on should be true", powerStatus.reverbOn);
    
    }
  }

  function toggleDelay() {
    console.log("delay");
    const options = { debug: true, delayTime: "4n", feedBack: 0.4 };
    const pingPong = new Tone.PingPongDelay(options).toDestination();
    toneMic.connect(pingPong);
    pingPong.connect(toneRecorder);
  }

  function toggleChours() {
    console.log("chours");
    const chorus = new Tone.Chorus(10, 0, 1).toDestination();
    toneMic.connect(chorus);
    chorus.connect(toneRecorder);
  }

  /**
   * https://tonejs.github.io/docs/14.7.77/Distortion
   * todo: research options overtone , input:gain, wet/dry to fix latency
   */
  function toggleDistortion() {
    console.log('distortion')
    const dist = new Tone.Distortion(1).toDestination();
    toneMic.connect(dist);
    dist.connect(toneRecorder);
  }
</script>

<!-- TODO 
  Pads could possilbe be different bg colors 
  like andy warhal 
  or pad machines that have different color bgs for each pad

  pad section to play song sections
-->

<main>
 

  <!--Row header logo settings etc -->
  <Container>
    <!-- background: rgb(102 133 161);
          color: rgb(172 192 211);
          <a target="_blank" href="https://icons8.com/icon/b0M9m0XIfOcD/radio-tower">Radio Tower</a> icon by <a target="_blank" href="https://icons8.com">Icons8</a>
   
      fav
      <a target="_blank" href="https://icons8.com/icon/b0M9m0XIfOcD/radio-tower">Radio Tower</a> icon by <a target="_blank" href="https://icons8.com">Icons8</a>
        -->

    <!-- Header -->
    <Row
      style="color: #5b87c2; font-size: x-large; height:200px; font-family: fantasy;margin-top:10px;"
    >
      <Col size="6">
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
      <Col>
        <Container>
          <button on:click={getMediaDevice} class="modal-ctrls--effects">start</button>
        </Container>
      </Col>
    </Row>
  </Container>

  <!-- Main Controls -->
  <Container>
    <!-- Mic Play -->
    <Row style="margin-bottom:10px">
      <Col size="5"></Col>
      <div class="app-ctrls--main">
        <!-- Mic -->
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
    </Row>

    <Row>
      <Col size="5"></Col>
      <div class="app-ctrls--main">
        <!-- Effects
          this could be mixer to enhance over all quality of recording and playback.
        -->
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
      <!-- <Col size="4"></Col> -->
    </Row>
  </Container>

  <!-- Reverb Delay-->
    <!-- SlOT -->
    <!-- <Effects> -->
      <div class:active style="margin-top: 10px;">
        <Container>
          <Row style="margin-bottom: 10px;">
            <!-- <Col size="6"></Col> -->
            <Col size="5"></Col>
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
          <!-- </Container> -->
  
          <!-- Chorus Distortion -->
          <!-- <Container style="max-width: 300px;"> -->
          <Row>
            <!-- <Col size="6"></Col> -->
            <Col size="5"></Col>
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
            <!-- <Col size="4" /> -->
          </Row>
        </Container>
      </div>
    <!-- </Effects> -->

  

  <!-- footer links contact etc. -->
</main>

<style>
  .active {
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

  .modal.s-wBJb4QHrfejj {
    padding-top: 10px !important;
    min-width: 300px !important;
    display: flex !important;
    align-items: center !important;
    background: #12395bed !important;
    top: 36% !important;
    left: 54% !important;
  }

  /* .background.s-wBJb4QHrfejj {
    background-color: none !important;
  } */

  .app-ctrls--main {
    padding-right: 10px;
  }



  .modal-ctrls--effects {
    color: #f5f0f0;
  }

  /* .powerStatusBtn {
    background: none !important;
    border: none !important;
    padding: unset !important;
  } */

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
