<script>
  // @ts-ignore
  import 'chota';
  // @ts-ignore
  import { Modal, Button, Card, Row, Col, Container, Input } from 'svelte-chota';
  // @ts-ignore
  
  let modal_open = false;

  let recordStatus = { isRecording: true }
  let playStatus = { isPlaying: true }
  let powerStatus = {
    reverbOn: true,
    delayOn: true,
    chorusOn: true,
    distortionOn: true
  }

  let revOn = '5a86c1';
  let revOff = 'red'

  /**
     * @param {any} ctrl
     */
  function togglePowerStatus(ctrl) {
    switch(ctrl) {
      case powerStatus.reverbOn:
        powerStatus.reverbOn = !powerStatus.reverbOn
        break;
      case powerStatus.delayOn:
        powerStatus.delayOn = !powerStatus.delayOn
        break;
      case powerStatus.chorusOn:
        powerStatus.chorusOn = !powerStatus.chorusOn
        break;
      case powerStatus.distortionOn:
        powerStatus.distortionOn = !powerStatus.distortionOn
        break;
}
  }

  function toggleRecordStatus() {
    recordStatus.isRecording = !recordStatus.isRecording
  }

  function togglePlayStatus() {
    playStatus.isPlaying = !playStatus.isPlaying
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
<Modal bind:open={modal_open}>
 
  <!-- Reverb -->
    <Card style="display:flex; justify-content: space-between;">
      <div>
        <!-- svelte-ignore a11y-missing-attribute -->
        <!-- <button class="powerStatusBtn" on:click={togglePowerStatus(powerStatus.reverbOn)}> -->
          {#if powerStatus.reverbOn}
            <button class="powerStatusBtn" on:click={togglePowerStatus(powerStatus.reverbOn)}>
              <img src="https://icongr.am/jam/power.svg?size=30&color=5a86c1" title="Turn Reverb On/Off" />
            </button>
            {/if} 
  
          {#if !powerStatus.reverbOn}
            <!-- svelte-ignore a11y-missing-attribute -->
            <button class="powerStatusBtn" on:click={togglePowerStatus(powerStatus.reverbOn)}>
              <img src="https://icongr.am/jam/power.svg?size=30&color=black" title="Turn Reverb On/Off" />
            </button>

          {/if}
        <!-- </button> -->

        <h6 class="modal-ctrls--effects">Reverb</h6>
      </div>
      <div style="align-self: center">
        <Input type="range" value="9" min="0" max="10"/>
      </div>
      <!-- svelte-ignore a11y-missing-attribute -->
      <div>
        <img src="https://icongr.am/jam/info.svg?size=20&color=5a86c1" title="Use slider to control the amount of Reverb"/>
      </div>
    </Card>

    <!-- Delay -->
    <Card style="display:flex; justify-content: space-between;">
      <div>
        <img src="https://icongr.am/jam/power.svg?size=30&color=5a86c1" />
        <h6 class="modal-ctrls--effects">Delay</h6>
      </div>
      <div style="align-self: center;">
        <Input type="range" value="9" min="0" max="10"/>
      </div>
      <div><img src="https://icongr.am/jam/info.svg?size=20&color=5a86c1" /></div>
    </Card>

    <!-- Chorus -->
    <Card style="display:flex; justify-content: space-between;">
      <div>
        <img src="https://icongr.am/jam/power.svg?size=30&color=5a86c1" />
        <h6 class="modal-ctrls--effects">Chorus</h6>
      </div>
      <div style="align-self: center;">
        <Input type="range" value="9" min="0" max="10"/>
      </div>
      <div><img src="https://icongr.am/jam/info.svg?size=20&color=5a86c1" /></div>
    </Card>

    <!-- Distortion -->
    <Card style="display:flex; justify-content: space-between;">
      <div>
        <img src="https://icongr.am/jam/power.svg?size=30&color=5a86c1" />
        <h6 class="modal-ctrls--effects">Distortion</h6>
      </div>
      <div style="align-self: center;">
        <Input type="range" value="9" min="0" max="10"/>
      </div>
      <div><img src="https://icongr.am/jam/info.svg?size=20&color=5a86c1" /></div>
    </Card>

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
    <Row style="color: #0d62ab; font-size: x-large; height:200px; font-family: fantasy;margin-top:10px;">
      <Col size="12">
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

  <!-- controls -->
  <Container>
    <!-- Mic Play -->
    <Row style="margin-bottom:10px">
      <Col size="5"></Col>

      <div class="app-ctrls--main">
        
        <!-- Mic -->
        <!-- svelte-ignore a11y-missing-attribute -->
        <button on:click={toggleRecordStatus}>
          {#if recordStatus.isRecording}
            <img src="https://icongr.am/jam/mic-f.svg?size=45&color=f5f0f0" />
            <!-- <img src="https://icongr.am/clarity/microphone.svg?size=45&color=f5f0f0" /> -->
          {/if}
  
          {#if !recordStatus.isRecording}
            <img src="https://icongr.am/jam/stop.svg?size=45&color=f5f0f0" />
          {/if}
        </button>

        <!-- Play -->
        <!-- svelte-ignore a11y-missing-attribute -->
        <button  on:click={togglePlayStatus}>
          {#if playStatus.isPlaying}
            <img src="https://icongr.am/clarity/play.svg?size=45&color=f5f0f0" />
          {/if}
  
          {#if !playStatus.isPlaying}
            <img src="https://icongr.am/jam/stop.svg?size=45&color=f5f0f0" />
          {/if}
        </button>
      </div>
    </Row>
  
    <Row>
      <Col size="5"></Col>
      <div class="app-ctrls--main">
        <!-- Effects -->
        <!-- svelte-ignore a11y-missing-attribute -->
        <button  on:click={event => modal_open = true}>
          <img src="https://icongr.am/entypo/sound-mix.svg?size=45&color=f5f0f0" />
        </button>

        <!-- Save -->
        <!-- svelte-ignore a11y-missing-attribute -->
        <button >
          <img src="https://icongr.am/feather/download-cloud.svg?size=45&color=f5f0f0" />
        </button>
      </div>
      <Col size="4"></Col>
   
    </Row>
  </Container>

  <!-- footer links contact etc. -->

</main>


<style>
  button {
    background-color: #5a86c1 !important;
    border: none;
  }

  .app-ctrls--main {
    padding-right: 10px;
  }

  .modal-ctrls--effects {
    color: #5a86c1;
  }

  .powerStatusBtn {
    background: none !important;
    border: none !important;
    padding: unset !important;
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
</style>
