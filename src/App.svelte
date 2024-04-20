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
  import Range from "./lib/Range.svelte"
  
  let value = 0;
  let value2 = 0;
  let value3 = 0;
  let value4 = 0;
  let chorusFrequency = 0;
  let chorusAmt = 0;
  let chorusDelay = 0;
  let chorusSpread = 0;
  let chorusDepth = 0;
  let toneRecorder = null;
  let toneMic;
  let toneContext;
  let toneRecordBlob = null;
  let player;
  let audioStream;
  let reverb = null;
  let revPre = 0;
  let revAmt = 0;
  let revDecay = 0;
  let autoFilter = null;
  let sw = null;
  let vib = null;
  let delay = null;
  let chorus = null;
  let phaser = null;
  let ps = null;
  let dirt = null;
  let active = true
  let activeCtrls = true
  let activeRadio = false
  let recordStatus = {
    isRecording: false,
    record: "https://icongr.am/jam/mic-f.svg?size=45&color=f5f0f0",
    stop: "https://icongr.am/jam/mic-f.svg?size=45&color=e2ef0b",
  };
  let recoredSrc = recordStatus.record;
  let playStatus = {
    isPlaying: false,
    playImgSrcOn:  "https://icongr.am/clarity/play.svg?size=45&color=e2ef0b",
    playImgSrcOff: "https://icongr.am/clarity/play.svg?size=45&color=f5f0f0",
    playImgSrc:    "https://icongr.am/clarity/play.svg?size=45&color=f5f0f0"
  };
  let powerStatus = {
    micOn: false,
    toneMicMute: true
  };
  let reverbStatus = {
    reverbOn: false,
    reverbImgSrcOn: "https://icongr.am/jam/power.svg?size=30&color=e2ef0b",
    reverbImgSrcOff: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0",
    reverbImgSrc: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0"
  }
  let delayStatus = {
    delayOn: false,
    delayImgSrcOn: "https://icongr.am/jam/power.svg?size=30&color=e2ef0b",
    delayImgSrcOff: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0",
    delayImgSrc: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0"
  }
  let chorusStatus = { 
    chorusOn: false,
    chorusImgSrcOn: "https://icongr.am/jam/power.svg?size=30&color=e2ef0b",
    chorusImgSrcOff: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0",
    chorusImgSrc: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0"
  }

  let dirtStatus = {
    dirtOn: false,
    dirtImgSrcOn: "https://icongr.am/jam/power.svg?size=30&color=e2ef0b",
    dirtImgSrcOff: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0",
    dirtImgSrc: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0"
  }
  
  $: if(powerStatus.micOn) {
    console.log('powerstatus tonecontext state', toneContext.state)
    console.log('powerstatus tonecontext state', toneContext)
    console.log('powerstatus micon', powerStatus.micOn)

  }

  /**
   * @description Suspend/Mute 
  */
  $: if(powerStatus.toneMicMute)  {
    if(toneMic) {
      console.log('powerstatus:: mic muted should be true:',powerStatus.toneMicMute)

      if((!reverbStatus.reverbOn && !recordStatus.isRecording && !delayStatus.delayOn && !chorusStatus.chorusOn && !dirtStatus.dirtOn && !playStatus.isPlaying) ) {
        toneContext.rawContext.suspend()
      }
    }
  } else if(powerStatus.toneMicMute === false) {
    /**
     * set up Svelet await syntax
     * await toneContext.rawcontext.resume()
    */
    if(toneMic) {
      console.log('powerstatus:: mic muted should be false', powerStatus.toneMicMute)
    }
  }

  /**
   * Distortion BitCrusher -WaveShaper 
   * this effect will shape current audio with distortion
   */
  function createWaveShaperDistorion() {
    console.log('tone context :::_context', Tone.context)
    let ctx = new AudioContext()
    let oscillator1 = new OscillatorNode(ctx);
    let bitCrusher = new WaveShaperNode(ctx)
    oscillator1.connect(bitCrusher).connect(ctx.destination)
    toneMic.connect(bitCrusher)

    oscillator1.start()

    // create samples, curve, levels
    let x;
    let n;
    let y;
    let nSamples = 65536;
    let curve = new Float32Array(nSamples);
    let nLevels = Math.pow(2, 3);
    for(n=0; n < nSamples; ++n) {
      x = nLevels/nSamples
      y = Math.floor(x)
      curve[n] = (2*y+1)/nLevels-1
    }
   
    bitCrusher.curve = curve

  }

  async function releaseMediaDevice() {
    if(toneMic) {
      toneMic.dispose();
    }
  }

  async function getMediaDevice() {
    activeCtrls = !activeCtrls
    activeRadio = !activeRadio

    /**
     * start the Audio Context state = 'running'
    */
    await Tone.start();

    if (!toneMic) {
      toneMic = new Tone.UserMedia({volume: -30, mute: true}).toDestination();
      powerStatus.micOn = true;
      toneContext = Tone.context;
      audioStream = await toneMic.open();
      console.log("mic", audioStream);
    }
  }

  function togglePlayStatus() {
    
    if (toneRecordBlob) {
      playStatus.isPlaying = !playStatus.isPlaying;
      if (playStatus.isPlaying) {
        playStatus.playImgSrc = playStatus.playImgSrcOn;
        powerStatus.toneMicMute = false
        play();
      } 
      else {
        playStatus.playImgSrc = playStatus.playImgSrcOff;
        stop();
      }
    }
    else {
      console.log('no audio')
    }
  }

  async function play() {
    toneRecordBlob
    .arrayBuffer()
    .then((arrayBuffer) => toneContext.decodeAudioData(arrayBuffer))
    .then(async (audioBuffer) => {
      console.log("blob", audioBuffer);

      player = new Tone.Player({ url: audioBuffer }).toDestination();

      if(toneContext.state === 'running') {
        console.log('player should be started', player.state)
        console.log('player should be running', player.context.state)
      } else {
        await Tone.context.resume()
        if(audioStream) {
          audioStream.volume.value = 0
        }
      }

      if(audioBuffer) {

        player.start();
      } 
      console.log('player', player)
    });
  }

  function stop() {
    player.stop();
    console.log("stop");
    toneContext.rawContext.suspend();

    if(player) {
      console.log('player should be stopped', player.state)
      console.log('player should be suspended', player.context.state)
      player.dispose()
    }

    console.log('player', player)


  }

  function toggleRecordStatus() {
    recordStatus.isRecording = !recordStatus.isRecording;

    if (recordStatus.isRecording) {
      recoredSrc = recordStatus.stop;
      console.log('recording')
      startRecord();
    } else {
      recoredSrc = recordStatus.record;
      stopRecord();
      powerStatus.toneMicMute = true
    }
  }

  async function startRecord() {
    await Tone.context.resume();
    if(toneContext.state === 'running') {
      console.log('tone context state::', 'running', toneContext.state)
    } else {
      console.log('tone context state::', 'stoppd', toneContext.state)
    }

    if(!toneRecorder) {
      toneRecorder = new Tone.Recorder();
      audioStream.volume.value = 0
      toneMic.connect(toneRecorder);
      powerStatus.toneMicMute = false

    }
    /**
     * toneRecorder.state = 'started/stopped'
     * toneRecorder.context.state = 'suspended/running'
    */
    console.log('recorder started', toneRecorder)

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
      ps.connect(toneRecorder);
      phaser.connect(toneRecorder)
      console.log('recording w/ chrs', chorus)
    }
    if(dirt) {
      dirt.connect(toneRecorder);
      console.log('recording w/ dirt', dirt)
    }

    await toneRecorder.start();
  }

  function shouldSuspendWhenEffectStatusOff() {
    // suspend if no effects 
    let shouldSuspend = false;
    if (!reverbStatus.reverbOn) {
      shouldSuspend = true
    }
    if (!chorusStatus.chorusOn) {
      shouldSuspend = true
    }
    if (!delayStatus.delayOn) {
      shouldSuspend = true
    }
    if (!dirtStatus.dirtOn) {
      shouldSuspend = true
    }
    return shouldSuspend;
  }

  async function stopRecord() {
    toneRecordBlob = await toneRecorder.stop();
    
    await toneMic.disconnect(toneRecorder)
    await toneRecorder.dispose()
    toneRecorder = null;
    
    console.log('recorder stopped:::', toneRecorder, ":::tonecontext::", toneContext.state)
  }

  function toggleReverbStatus() {
    if (reverbStatus.reverbOn) {
      reverbStatus.reverbOn = false;
      powerStatus.toneMicMute = true;
      reverbStatus.reverbImgSrc = reverbStatus.reverbImgSrcOff;
      disconnectReverb()
      console.log("reverb status on should be false", reverbStatus.reverbOn);
    } else {
      reverbStatus.reverbOn = true;
      reverbStatus.reverbImgSrc = reverbStatus.reverbImgSrcOn;
      powerStatus.toneMicMute = false;
      connectReverb()
      console.log("reverb status on should be true", reverbStatus.reverbOn);
    }
  }

  async function createReverb() {
    if (!reverb || reverb['_wasDisposed'] === true) {
      console.log('start reverb')
      reverb = new Tone.Reverb({
        wet: revAmt,
        decay: revDecay,
        preDelay: revPre
      }).toDestination();
    }
  }

  /**
   * Reverb is spilling into recording after reverb is stopped and a new recording begins 
   */
  async function connectReverb() {
    await Tone.context.resume()
    console.log('...connecting reverb')
    if(toneContext.state === 'running') {
      if(audioStream) {
        audioStream.volume.value = 0
      }

      toneMic.connect(reverb);

      console.log('tone context running = ', toneContext.state)
      console.log('reverb should be running', reverb.context.state)
    } 

    console.log('reverb node', reverb)
  }

  function disconnectReverb() {
    toneMic.disconnect(reverb)
    reverb.dispose()
    if(reverb['_wasDisposed'] === true) {
      console.log('reverb was dispposed')
    }
    reverb = null

    console.log('reverb node', reverb)
  }

  async function toggleReverb() {
    createReverb()
    toggleReverbStatus()
  }

  async function toggleDelay() {
    if(delayStatus.delayOn) {
      delayStatus.delayOn = false
      delayStatus.delayImgSrc = delayStatus.delayImgSrcOff;
      powerStatus.toneMicMute = true;
    } else {
      delayStatus.delayImgSrc = delayStatus.delayImgSrcOn;
      delayStatus.delayOn = true
      powerStatus.toneMicMute = false;
    }

    if(!delay || delay['_wasDisposed'] === true) {
      
      await Tone.context.resume()
      powerStatus.toneMicMute = false;
      if(audioStream) {
        audioStream.volume.value = 0
      }
      const options = { debug: true, delayTime: "4n", feedBack: 0.4 };
      delay = new Tone.PingPongDelay(options).toDestination();
      toneMic.connect(delay);
      console.log("delay", delay, "mic should muted false: ", toneMic.mute);
    } else {
      powerStatus.toneMicMute = true;
      delay.disconnect()
      delay.dispose()
      delay = null;
      console.log("delay", delay, "mic should muted true: ", toneMic.mute);
    }
  }

  async function toggleChours() {
    if(chorusStatus.chorusOn) {
      chorusStatus.chorusOn = false
      chorusStatus.chorusImgSrc = chorusStatus.chorusImgSrcOff;
      powerStatus.toneMicMute = true;
    } else {
      chorusStatus.chorusImgSrc = chorusStatus.chorusImgSrcOn;
      chorusStatus.chorusOn = true
      powerStatus.toneMicMute = false;
    }

    if(!chorus || chorus['_wasDisposed'] === true) {
      await Tone.context.resume()
      powerStatus.toneMicMute = false;
      if(audioStream) {
          audioStream.volume.value = 0
        }
      
      const options = { frequency: chorusFrequency, delayTime: chorusDelay, depth: chorusDepth, wet: chorusAmt, spread: chorusSpread };
      chorus = new Tone.Chorus(options).toDestination();

      const autoFilter = new Tone.AutoFilter("1n").toDestination().start();

      console.log("chours", chorus);

    } else {
      powerStatus.toneMicMute = true;
      chorus.disconnect()
      chorus.dispose()
      chorus = null;
      console.log("chours", chorus);
    }
  }

  /**
   * https://tonejs.github.io/docs/14.7.77/Distortion
   */
  async function toggleDirt() {
    if(dirtStatus.dirtOn) {
      dirtStatus.dirtOn = false
      dirtStatus.dirtImgSrc = dirtStatus.dirtImgSrcOff;
      powerStatus.toneMicMute = true;
    } else {
      dirtStatus.dirtImgSrc = dirtStatus.dirtImgSrcOn;
      dirtStatus.dirtOn = true
      powerStatus.toneMicMute = false;
    }

    if(!dirt || dirt['_wasDisposed'] === true) {
      const options = { oversample: "4x", distortion: 1}
      dirt = new Tone.Distortion(options).toDestination();
      

      await Tone.context.resume()
      console.log('...connecting dirt')
      if(toneContext.state === 'running') {
        if(audioStream) {
          audioStream.volume.value = 0
        }
      }
      toneMic.connect(dirt);
      console.log('distortion', dirt)
    } else {
      dirt.disconnect()
      dirt.dispose()
      dirt = null;
      console.log("distortion", dirt);
    }
  }

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


  function slideParam(p) {
    console.log('param before::', chorusFrequency, 'p::::0', p)

    chorusFrequency = p.target.value
    console.log('param::', chorusFrequency, 'p::::0', p)
  }
  function slideParam2(p) {
    chorusDelay = p.target.value
    console.log('param::', chorusDelay, 'p::::1', p)
  }
  function slideParam3(p) {
    chorusAmt = p.target.value
    console.log('param::', chorusAmt, 'p::::2', p)
  }
  function slideParam4(p) {
    chorusSpread = p.target.value
    console.log('param::', chorusSpread, 'p::::3', p)
  }
  function slideParam5(p) {
    chorusDepth = p.target.value
    console.log('param::', chorusDepth, 'p::::4', p)
  }


  function slideParamRevDec(p) {
    console.log('param before::', revDecay, 'p::::0', p)
  
    revDecay = p.target.value
    console.log('param reverb decay::', revDecay, 'p::::0', p)
  }

  function slideParamRevPre(p) {
    console.log('param before::', revPre, 'p::::0', p)
   
    revPre = p.target.value
    console.log('param reverb revPre::', revPre, 'p::::0', p)
  }

  function slideParamRevAmt(p) {
    console.log('param before::', revAmt, 'p::::0', p)
   
    revAmt = p.target.value
    console.log('param reverb revAmt::', revAmt, 'p::::0', p)
  }
</script>

<Container>
  <main>
    <!-- Top Bar -->
    <Container>
      <Row style="color: #6babff; font-size: x-large; height:200px; font-family: fantasy;margin-top:10px;">
        <Col size="6">
          <!-- Logo -->
          <div
            style="display: flex;
            align-items: center;"
          >
            <!-- <h6>Song Pad</h6> -->
            <!-- svelte-ignore a11y-missing-attribute -->
            <!-- https://icongr.am/jam/station.svg?size=26&color=acc0d3 -->
            <button style="background: #ffffff !important; max-width: initial !important; display: flex !important; align-items: center !important; color: rgb(107, 171, 255);">
              <!-- svelte-ignore a11y-missing-attribute -->
              Song Pad
              <img src="public\icons8-radio-tower-48.png" />
            </button>
            <button style="background: none;" on:click={getMediaDevice}>start</button>
            <button style="background: none;" on:click={releaseMediaDevice}>stop</button>
            <!-- <button style="background: none;" on:click={createWaveShaperDistorion}>bitcrusher</button> -->
            <!-- <img src="public\icons8-radio-tower-48.png" /> -->
          </div>
        </Col>
        <!-- Start Mic -->
        <!-- {#if !activeRadio} -->
        <Container>
          <Row style="margin-bottom: 20px">
            <Col size="5"></Col>
          </Row>
        </Container>
        <!-- {/if} -->
      </Row>
    </Container>
  
    <div class="fox">
      <!-- Main Controls -->
    <Container>
      <Row>
      <!-- <Row style="margin-bottom:10px;"> -->
        <Col size="5"></Col>
          <div class="app-ctrls--main space">
            <!-- Record -->
            <!-- svelte-ignore a11y-missing-attribute -->
            <button on:click={toggleRecordStatus}>
              <img src={recoredSrc} />
            </button>
    
            <!-- Play -->
            <!-- svelte-ignore a11y-missing-attribute -->
            <button on:click={togglePlayStatus}>
              <img src={playStatus.playImgSrc} />
            </button>
          </div>
      </Row>
  
      <Row>
        <Col size="5"></Col>
          <div class="app-ctrls--main space">
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
      </Row>
    </Container>
  
    <!-- Effects -->
    {#if !active}
    <Container>
      <div class:active transition:fade>
      <!-- <div class:active style="margin-top: 10px;" transition:fade> -->
        <!-- <Row style="margin-bottom: 10px;"> -->
        <!-- Reverb Delay-->
        <Row>
          <Col size="5"></Col>
         
          <div class="app-ctrls--main space">
            <!-- svelte-ignore a11y-missing-attribute -->
           <div>
            <button on:click={toggleReverb}>
              <img src={reverbStatus.reverbImgSrc} title="Turn Reverb On/Off" />
              <h6 class="modal-ctrls--effects">Reverb</h6>
             
              <!-- <Range on:change={slideParam} id="basic-slider" />
              <Range on:change={slideParam2} id="basic-slider2" />
              <Range on:change={slideParam3} id="basic-slider3" />
              <Range on:change={slideParam4} id="basic-slider4" /> -->
            </button>
            <div>
              <input type="range" id="chF" name="chF"
                   step=".01"  min="0" max=".20" bind:value={revPre} on:input={slideParamRevPre}>
            </div>
            <div>
              <input type="range" id="chD" name="chD"
                   step=".5"  min="0" max="5" bind:value={revDecay} on:input={slideParamRevDec}>
            </div>
            <div>
              <input type="range" id="chD" name="chD"
                   step=".1"  min="0" max="1" bind:value={revAmt} on:input={slideParamRevAmt}>
            </div>
           </div>
  
           
            <div>
              <button on:click={toggleDelay}>
                <!-- svelte-ignore a11y-missing-attribute -->
                <img src={delayStatus.delayImgSrc} />
                <h6 class="modal-ctrls--effects">Delay</h6>
              </button>
              <!-- <div>
                <input type="range" id="chF" name="chF"
                     step=".1"  min="0" max="500" bind:value={revPre} on:input={slideParamRevPre}>
              </div>
              <div>
                <input type="range" id="chD" name="chD"
                     step=".1"  min="0" max="500" bind:value={revDecay} on:input={slideParamRevDec}>
              </div> -->
            </div>
          </div>
        </Row>
  
        <!-- Chorus Distortion -->
        <Row>
          <Col size="5"></Col>
          
          <div class="app-ctrls--main">
           <div>
            <button on:click={toggleChours}>
              <!-- svelte-ignore a11y-missing-attribute -->
              <img src={chorusStatus.chorusImgSrc} />
              <h6 class="modal-ctrls--effects">Chorus</h6>
             
              <!-- <Range on:change={slideParam} id="basic-slider" />
              <Range on:change={slideParam2} id="basic-slider2" />
              <Range on:change={slideParam3} id="basic-slider3" />
              <Range on:change={slideParam4} id="basic-slider4" /> -->
            </button>
            <div>
              <input type="range" id="chF" name="chF"
                   step=".1"  min="0" max="10" bind:value={chorusFrequency} on:input={slideParam}>
            </div>
            <div>
              <input type="range" id="chD" name="chD"
                     min="0" max="20" bind:value={chorusDelay} on:input={slideParam2}>
            </div>
            <div>
              <input type="range" id="chS" name="chA"
                    step=".1" min="0" max="1" bind:value={chorusAmt} on:input={slideParam3}>
            </div>
            <div>
              <input type="range" id="chA" name="chA"
                    step="20" min="0" max="180" bind:value={chorusSpread} on:input={slideParam4}>
            </div>
            <div>
              <input type="range" id="chA" name="chA"
                    step=".1" min="0" max="1" bind:value={chorusDepth} on:input={slideParam5}>
            </div>
           </div>
  
           <div>
            <button on:click={toggleDirt}>
              <!-- svelte-ignore a11y-missing-attribute -->
              <img src={dirtStatus.dirtImgSrc} />
              <h6 class="modal-ctrls--effects">Dirt</h6>
            </button>
            <!-- <div>
              <input type="range" id="chF" name="chF"
                   step=".1"  min="0" max="500" bind:value={revPre} on:input={slideParamRevPre}>
            </div>
          </div>
            <div>
              <input type="range" id="chD" name="chD"
                   step=".1"  min="0" max="500" bind:value={revDecay} on:input={slideParamRevDec}>
            </div> -->
          </div>
        </Row>
      </div>
    </Container>
    {/if}
    </div>
  
  </main>
</Container>


<style>



  @media screen and (  max-width: 1200px) and (min-width: 599px) {
    .space {
      margin-bottom: 5px;
    }
  }
  .active {
    display: none;
  }

  @media screen and (max-width: 598px) {
    .fox {
      margin-left: 25%;
    }
    .space {
      margin-bottom: 0px !important;
    }
  }

  .activeRadio {
    display: none;
  }
  .button,
  input[type="button"],
  input[type="reset"],
  input[type="submit"],
  button {
    /* background-color: #6684ff !important; */
    border: none;
    min-width: 95px;
    max-width: 95px;
  }
  .app-ctrls--main {
    padding-right: 10px;
    /* dmargin-bottom: 5px !important; */
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
    --grid-gutter: 1rem;
    --font-size: 1.6rem;
    --font-color: #333333;
    --font-family-sans: sans-serif;
    --font-family-mono: monaco, "Consolas", "Lucida Console", monospace;
  }

  /* theme colors
  467eaf  #1a9ce2  #35bdec  #50e5fe  #199be2 */

</style>
