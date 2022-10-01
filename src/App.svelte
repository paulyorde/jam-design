<script>
  // @ts-ignore
  import 'chota';
  // @ts-ignore
  import { Modal, Button, Card, Row, Col, Container, Input } from 'svelte-chota';
  import * as Tone from 'tone'
  import { saveAs } from 'file-saver'
  import * as encoder from 'audio-encoder'
  // import * as encoder from './assets/audioEncoder'
  // const encoder = require('audio-encoder')
  // import * as encoder from 'audio-encoder'


  // these object porperties below could be here in main file passed into each component
  // or used as a store?
  // usermedia , context happens in main file
  // recorder in recorder, player in player, etc...
  
  let toneRecorder
  let toneMic
  let toneContext
  let toneStreamNode
  let toneStreamSourceNode
  let toneRecordBlob
  let player
  let atcStream

  //   const _reverb = new Tone.Reverb({"wet": 1,"decay": 1.9,"preDelay": 1.00})
    // const options = {debug: true, delayTime: "4n", feedBack: .04}
    // const _pingPong =  new Tone.PingPongDelay({debug: true, delayTime: "4n", feedBack: .04})

//467eaf  #1a9ce2  #35bdec  #50e5fe  #199be2


     // functional = data in - data out 
     // 
    async function getMediaDevice() {
      await Tone.start()
      console.log('before toneMic', toneMic)
      if(!toneMic) {
        // pass in { toneMic: userMedia, toneContext: audioContext } 
        // -> returns copy with updated userMedia to calling method.
       toneMic = new Tone.UserMedia().toDestination();
       toneContext = Tone.context;
       toneRecorder = new Tone.Recorder();

      //  toneMic.open().then(stream => {
      //   console.log('mic',stream)
      //   atcStream = stream
      //  });
      }
      console.log('arter toneMic', toneMic)


      /**
        * stream state = started
        * toneContext/audioContext = running
      */
      startRecord()
    }

    // pass in userMedia object 
    async function startRecord() {
      console.log('start record stream', atcStream)
      // if !mic
      toneMic.open().then((stream) => {
        console.log('mic',stream)
        atcStream = stream
        // toneMic.mute = false
      });
      // is a new recorder for every mic open neccesary 
      // instead just disconnect from recorder created on create user media which happens once
      // toneRecorder = new Tone.Recorder();
      // try moving to after stream opens 
      toneMic.connect(toneRecorder);
      await Tone.context.resume();
      
      await toneRecorder.start()
    
      // toneStreamNode = toneContext.createMediaStreamDestination();
      // toneStreamSourceNode = toneContext.createMediaStreamSource(toneStreamNode.stream);
  }


  // pass in recorder, mic, context.
  async function stopRecord() {                                                                         
    toneRecordBlob = await toneRecorder.stop()
    console.log('stop record stream ', atcStream)
    console.log('stop record tonecontext::', toneContext)
    /**
     * Cleanup
    */
    await toneMic.disconnect(toneRecorder)
    Tone.context.dispose()
    toneContext.rawContext.suspend()
    toneMic.close()
    // toneRecorder.disconnect()
    /**
     * dispose the Stream to get it to stop or mute mic
     * how to mute mic = mic.mute = true or setMic(true) or destination.mute (speakers)
     * how to get context state to stopped / suspended = audioCtx.suspend (web api)
    */
    // toneRecorder.dispose()
    // toneContext


  }
  

  async function play() {
    toneRecordBlob.arrayBuffer()
    // decode is expensive - does tone make it faster - can this passed to worker
    .then(arrayBuffer => toneContext.decodeAudioData(arrayBuffer))
    .then(async audioBuffer => {
      console.log('blob', audioBuffer)
      await Tone.context.resume();

      // not need to be new player each time.
      player = new Tone.Player({url: audioBuffer}).toDestination()
      player.start()
    })
  }

  function stop() {
    player.stop()
    console.log('stop')
    toneContext.rawContext.suspend()
  }

  /**
   * V2
   * download to computer 
   * OR
   * dowload to track pad (vrs, chrs, bridge, etc)
   */
  async function download() {
    console.log('blob')
    toneRecordBlob.arrayBuffer()
    .then(arrayBuffer => toneContext.decodeAudioData(arrayBuffer))
    .then(audioBuffer => {
     console.log('blob', audioBuffer)

      // @ts-ignore
    encoder(audioBuffer, 'WAV', (v) => console.log('happeing now', v), (blob) => {
    saveAs(blob, 'sound.mp3')
  })
    })
   
}
  
  // function downloadAudio() {
  //   console.log('download', toneRecordBlob.arrayBuffer)
  //   toneRecordBlob.arrayBuffer()
  //   .then(arrayBuffer => toneContext.decodeAudioData(arrayBuffer))
  //   .then(audioBuffer => {
  //     const anchor = document.createElement("a");
  //     anchor.download = "recording.webm";
  //     anchor.href = audioBuffer;
  //     anchor.click();
  //   })
    
  // }
  
  let modal_open = false;
  let recordStatus = { 
    isRecording: false,
    record: "https://icongr.am/jam/mic-f.svg?size=45&color=f5f0f0",
    // record: "https://icongr.am/jam/mic-f.svg?size=45&color=f5f0f0",
    stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0"
    // src: 
    // stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0"
  }
  let recoredSrc =  recordStatus.record
  let playStatus = { 
    isPlaying: false,
    play: "https://icongr.am/clarity/play.svg?size=45&color=f5f0f0",
    stop: "https://icongr.am/jam/stop.svg?size=45&color=f5f0f0"
  }
  let powerStatus = {
    reverbOn: true,
    delayOn: true,
    chorusOn: true,
    distortionOn: true,

    reverbStart: "https://icongr.am/jam/power.svg?size=30&color=f5f0f0",
    reverbStop: "https://icongr.am/jam/power.svg?size=30&color=black"
    
  }
  let playSrc =  playStatus.play
  let reverbSrc = powerStatus.reverbStart

  function togglePowerStatus(ctrl) {
    switch(ctrl) {
      case powerStatus.reverbOn:
        toggleReverb()
        console.log('reverb', powerStatus.reverbOn)
        powerStatus.reverbOn = !powerStatus.reverbOn
        if (powerStatus.reverbOn) {
          reverbSrc = powerStatus.reverbStart
        } else {
          reverbSrc = powerStatus.reverbStop
        }
        break;
      case powerStatus.delayOn:
        toggleDelay()
        powerStatus.delayOn = !powerStatus.delayOn
        break;
      case powerStatus.chorusOn:
        toggleChours()
        powerStatus.chorusOn = !powerStatus.chorusOn
        break;
      case powerStatus.distortionOn:
        toggleDistortion()
        powerStatus.distortionOn = !powerStatus.distortionOn
        break;
    }
  }

  function toggleRecordStatus() {
    recordStatus.isRecording = !recordStatus.isRecording
    if (recordStatus.isRecording) {
      recoredSrc = recordStatus.stop
      getMediaDevice()
      toneMic.mute = false
    } else {
      recoredSrc = recordStatus.record;
      stopRecord()
      toneMic.mute = true
    }
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

  /**
   * todo 
   * start effects first , then start record.
   */
  function toggleReverb() {
    const reverb = new Tone.Reverb({"wet": 1,"decay": 1.9,"preDelay": 0}).toDestination()
    toneMic.connect(reverb)
    reverb.connect(toneRecorder)
  }

  function toggleDelay() {
    console.log('delay')
    const options = {debug: true, delayTime: "4n", feedBack: .4}
    const pingPong =  new Tone.PingPongDelay(options).toDestination()
    toneMic.connect(pingPong)
    pingPong.connect(toneRecorder)
  }

  function toggleChours() {
    console.log('chours')
    const chorus = new Tone.Chorus(10, 0, 1).toDestination()
    toneMic.connect(chorus)
    chorus.connect(toneRecorder)
  }

  /**
   * https://tonejs.github.io/docs/14.7.77/Distortion
   * todo: research options overtone , input:gain, wet/dry to fix latency
   */
  function toggleDistortion() {
    const dist = new Tone.Distortion(1).toDestination();
    toneMic.connect(dist)
    dist.connect(toneRecorder)
  }

</script>



<!-- TODO 
  Pads could possilbe be different bg colors 
  like andy warhal 
  or pad machines that have different color bgs for each pad

  pad section to play song sections
-->

<main>
  <!-- Effect Controls -->
  <Modal class="modal-m" bind:open={modal_open}>
      <!-- Reverb Delay-->
      <!-- <Card style="display:flex; justify-content: space-between;"> -->
      <Container style="max-width:300px">
        <Row style="margin-bottom:10px">
          <!-- <Col size="6"></Col> -->
          <Col size=12>
            <div class="app-ctrls--main foo">
              <!-- svelte-ignore a11y-missing-attribute -->
              <button class="powerStatusBtn foo" on:click={toggleReverb} >
                <img src={reverbSrc} title="Turn Reverb On/Off" />
                <h6 class="modal-ctrls--effects">Reverb</h6>
              </button>

              <button on:click={toggleDelay} class="powerStatusBtn foo" >
                <!-- svelte-ignore a11y-missing-attribute -->
                <img src="https://icongr.am/jam/power.svg?size=30&color=f5f0f0" />
                <h6 class="modal-ctrls--effects">Delay</h6>
              </button>
            </div>
          </Col>
        </Row>
      </Container>

      <!-- Chorus Distortion -->
      <Container style="max-width: 300px;">
        <Row style="margin-bottom:10px">
          <!-- <Col size="6"></Col> -->
          <Col size=12>
            <div class="app-ctrls--main foo">
              <button on:click={toggleChours} class="powerStatusBtn foo" >
                <!-- svelte-ignore a11y-missing-attribute -->
                <img src="https://icongr.am/jam/power.svg?size=30&color=5a86c1" />
                <h6 class="modal-ctrls--effects">Chorus</h6>
              </button>
  
              <button on:click={toggleDistortion} class="powerStatusBtn foo" >
                <!-- svelte-ignore a11y-missing-attribute -->
                <img src="https://icongr.am/jam/power.svg?size=30&color=5a86c1" />
                <h6 class="modal-ctrls--effects">Distortion</h6>
              </button>
            </div>
          </Col>
        </Row>
      </Container>
    
  </Modal>


  <!--Row header logo settings etc -->
  <Container>
    <!-- background: rgb(102 133 161);
          color: rgb(172 192 211);
          <a target="_blank" href="https://icons8.com/icon/b0M9m0XIfOcD/radio-tower">Radio Tower</a> icon by <a target="_blank" href="https://icons8.com">Icons8</a>
   
      fav
      <a target="_blank" href="https://icons8.com/icon/b0M9m0XIfOcD/radio-tower">Radio Tower</a> icon by <a target="_blank" href="https://icons8.com">Icons8</a>
        -->

    <!-- Header -->
    <Row style="color: #5b87c2; font-size: x-large; height:200px; font-family: fantasy;margin-top:10px;">
      <Col size="6">
          <div style="display: flex;
        align-items: center;">
          <h6>Song Pad</h6>
          <!-- svelte-ignore a11y-missing-attribute -->
          <!-- https://icongr.am/jam/station.svg?size=26&color=acc0d3 -->
          <img  src="public\icons8-radio-tower-48.png" />
        </div>
        
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
          <img src={recoredSrc}  />
        </button>

        <!-- Play -->
        <!-- svelte-ignore a11y-missing-attribute -->
        <button  on:click={togglePlayStatus}>
          <img src={playSrc}  />
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
        <button  on:click={event => modal_open = true}>
          <img src="https://icongr.am/entypo/sound-mix.svg?size=45&color=f5f0f0" />
        </button>

        <!-- Save -->
        <!-- svelte-ignore a11y-missing-attribute -->
        <button on:click={download}>
          <img src="https://icongr.am/feather/download-cloud.svg?size=45&color=f5f0f0" />
        </button>
      </div>
      <!-- <Col size="4"></Col> -->
    </Row>
  </Container>

  <!-- Reverb Delay-->
      <!-- <Card style="display:flex; justify-content: space-between;"> -->
        <Container>
          <Row style="margin-bottom:10px">
            <!-- <Col size="6"></Col> -->
            <Col size="5"></Col>
              <div class="app-ctrls--main">
                <!-- svelte-ignore a11y-missing-attribute -->
                <button on:click={toggleReverb}>
                  <img src={reverbSrc} title="Turn Reverb On/Off" />
                  <h6 class="modal-ctrls--effects">Reverb</h6>
                </button>
  
                <button on:click={toggleDelay}>
                  <!-- svelte-ignore a11y-missing-attribute -->
                  <img src="https://icongr.am/jam/power.svg?size=30&color=f5f0f0" />
                  <h6 class="modal-ctrls--effects">Delay</h6>
                </button>
              </div>
          </Row>
        <!-- </Container> -->
  
        <!-- Chorus Distortion -->
        <!-- <Container style="max-width: 300px;"> -->
          <Row style="margin-bottom:10px">
            <!-- <Col size="6"></Col> -->
            <Col size="5"></Col>
              <div class="app-ctrls--main foo">
                <button on:click={toggleChours}>
                  <!-- svelte-ignore a11y-missing-attribute -->
                  <img src="https://icongr.am/jam/power.svg?size=30&color=f5f0f0" />
                  <h6 class="modal-ctrls--effects">Chorus</h6>
                </button>
    
                <button on:click={toggleDistortion}>
                  <!-- svelte-ignore a11y-missing-attribute -->
                  <img src="https://icongr.am/jam/power.svg?size=30&color=f5f0f0" />
                  <h6 class="modal-ctrls--effects">Distortion</h6>
                </button>
              </div>
              <Col size="4"></Col>
          </Row>
        </Container>

  <!-- footer links contact etc. -->

</main>


<style>
  .button, input[type='button'], input[type=reset], input[type=submit], button  {
    background-color: #5a86c1 !important;
    border: none;
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

  .foo {
    margin:10px;
    background: none !important;
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
</style>
