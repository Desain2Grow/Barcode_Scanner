<script>
  import { Html5Qrcode } from 'html5-qrcode'
  import { onMount } from 'svelte'
  import { writable } from 'svelte/store'

  let scanning = false
  let scannedValue = ""
  let recordedValues = writable([])

  let html5Qrcode

  onMount(init)

  function init() {
    html5Qrcode = new Html5Qrcode('reader')
  }

  function start() {
    html5Qrcode.start(
      { facingMode: 'environment' },
      {
        fps: 10,
        qrbox: { width: 250, height: 250 },
      },
      onScanSuccess,
      onScanFailure
    )
    scanning = true
  }

  async function stop() {
    await html5Qrcode.stop()
    scanning = false
  }

  function onScanSuccess(decodedText, decodedResult) {
    scannedValue = decodedText
  }

  function onScanFailure(error) {
    //console.warn(`Code scan error = ${error}`)
  }

  function recordScannedValue() {
    recordedValues.update(arr => [...arr, scannedValue])
    scannedValue = ""
  }

  function clearRecordedValues() {
    recordedValues.set([])
  }

  function copyToClipboard() {
    const textToCopy = $recordedValues.join('\n')
    navigator.clipboard.writeText(textToCopy)
      .then(() => console.log('Text copied to clipboard'))
      .catch((err) => console.error('Error copying text to clipboard:', err))
  }
</script>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 20px;
  }
  reader {
    width: 100%;
    min-height: 500px;
    background-color: black;
  }
  .recorded-values {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-auto-rows: minmax(50px, auto);
    grid-gap: 10px;
  }
  .recorded-value {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 5px;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #fff;
  }
  .buttons {
    display: flex;
    justify-content: space-around;
    gap: 20px;
    margin-top: 20px;
    width: 100%;
  }
</style>

<main>
  <reader id="reader"/>
  {#if scanning}
    <button on:click={stop}>stop</button>
  {:else}
    <button on:click={start}>start</button>
  {/if}

  <div>
    <input type="text" bind:value={scannedValue} placeholder="Scanned value" />
    <button on:click={recordScannedValue}>Record</button>
  </div>

  <div class="recorded-values">
    {#each $recordedValues.slice().reverse() as value, i}
      <div class="recorded-value">{value}</div>
    {/each}
  </div>

  <div class="buttons">
    <button on:click={clearRecordedValues}>Clear all</button>
    <button on:click={copyToClipboard}>Copy</button>
  </div>
</main>
