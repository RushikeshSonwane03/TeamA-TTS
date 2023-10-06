<script context="module">
  export { onMount } from 'svelte';
</script>

<script>
  import { onMount } from 'svelte';

  let synthesis;
  let voices = [];
  let synthesisVoice = '';
  let pitch = 1;
  let rate = 1;
  let volume = 1;
  let mediaRecorder;
  let audioChunks = [];
  let textToSpeak = '';
  let isFirstClick = true; // Flag to track the first click
  let isRecording = false; // Flag to track the recording state

  function getVoices() {
  return new Promise(resolve => {
    synthesis = window.speechSynthesis;
    synthesis.onvoiceschanged = () => {
      voices = synthesis.getVoices();
      
      // Log available voices to the console for debugging
      console.log(voices);
      
      // Filter out problematic voices or set a default voice as fallback
      voices = voices.filter(voice => !voice.name.includes('Problematic Voice')); // Replace 'Problematic Voice' with the actual name causing issues
      
      resolve();
    };
  });
}

  onMount(async () => {
    await getVoices();
  });

  function startRecording() {
    audioChunks = [];
    mediaRecorder.start();
    isRecording = true; // Set recording state to true when starting the recording
  }

  function stopRecording() {
    if (isRecording) {
      mediaRecorder.stop();
      mediaRecorder.onstop = () => {
        const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
        const audioUrl = URL.createObjectURL(audioBlob);
        const downloadLink = document.createElement('a');
        downloadLink.href = audioUrl;
        downloadLink.download = 'text-to-speech.wav';
        downloadLink.click();
        isRecording = false; // Set recording state to false after stopping the recording
        isFirstClick = true; // Reset isFirstClick flag to true after stopping the recording
      };
    }
  }

  function speak() {
    const utterance = new SpeechSynthesisUtterance(textToSpeak);
    utterance.pitch = pitch;
    utterance.rate = rate;
    utterance.volume = volume;
    const selectedVoice = voices.find(voice => voice.name === synthesisVoice);
    if (selectedVoice) {
      utterance.voice = selectedVoice;
    }

    synthesis.speak(utterance);
  }

  function setVoice(event) {
    synthesisVoice = event.target.value;
  }

  function setPitch(event) {
    pitch = parseFloat(event.target.value);
  }

  function setRate(event) {
    rate = parseFloat(event.target.value);
  }

  function setVolume(event) {
    volume = parseFloat(event.target.value);
  }

  onMount(() => {
    getVoices();

    // Initialize mediaRecorder for audio recording
    if ('MediaRecorder' in window) {
      navigator.mediaDevices.getUserMedia({ audio: true })
        .then(stream => {
          mediaRecorder = new MediaRecorder(stream);
          mediaRecorder.ondataavailable = event => {
            if (event.data.size > 0) {
              audioChunks.push(event.data);
            }
          };
        })
        .catch(error => {
          console.error('Error accessing microphone:', error);
        });
    } else {
      console.error('MediaRecorder not supported in this browser.');
    }
  });


  function speakAndRecord() {
    if (!isFirstClick) {
      speak(); // Speak the text only if it's not the first click
    }

    if (isFirstClick) {
      startRecording(); // Start audio recording only on the first click
      speak();
      isFirstClick = false; // Set the flag to false after the first click
    }
  }

</script>

<main>
  <div class="header">
    <i class="fas fa-volume-up" style="font-size: 36px; color: purple"></i>
    <h1 style="color: purple">Text-to-Speech Application</h1>
  </div>
  <div class="controls">
    <label for="pitch">Pitch:</label>
    <input type="range" min="0.1" max="2" step="0.1" bind:value={pitch} on:input={setPitch} id="pitch" />
    <br>
    <br>
    <label for="rate">Rate:</label>
    <input type="range" min="0.1" max="2" step="0.1" bind:value={rate} on:input={setRate} id="rate" />
    <br>
    <br>
    <label for="volume">Volume:</label>
    <input type="range" min="0" max="1" step="0.1" bind:value={volume} on:input={setVolume} id="volume" />
    <br>
    <br>
  </div>
  
  <textarea bind:value={textToSpeak} placeholder="Enter text to speak..." style="width: 100%; height: 150px; margin-bottom: 10px;"></textarea>
  
  <select on:change={setVoice} style="width: 100%; padding: 10px; border-radius: 5px; border: 1px solid #ccc; font-size: 16px; background-color: white; cursor: pointer; margin-bottom: 20px;">
    {#each voices as voice}
      <option value={voice.name}>{voice.name}</option>
    {/each}
  </select>

  <button on:click={speakAndRecord} style="background-color: purple; color: white; padding: 14px 20px; border: none; cursor: pointer; border-radius: 5px; font-size: 16px; margin-right: 10px;">Speak</button>

  <br>
  <br>
  <button on:click={stopRecording} style="background-color: purple; color: white; padding: 14px 20px; border: none; cursor: pointer; border-radius: 5px; font-size: 16px;">Save Audio</button>
</main>

<style>
  main {
    text-align: center;
    padding: 20px;
    background-color: #f5f5f5;
    border-radius: 10px;
    box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.2);
    width: 80%;
    max-width: 400px;
    margin: 0 auto;
  }

  h1 {
    font-size: 24px;
    margin-bottom: 20px;
  }

  textarea, select {
    width: 100%;
    padding: 10px;
    border-radius: 5px;
    border: 1px solid #ccc;
    font-size: 16px;
    background-color: white;
    margin-bottom: 20px;
    resize: none;
  }

  button {
    cursor: pointer;
    font-size: 18px;
    border: none;
    border-radius: 5px;
    padding: 14px 20px;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #45a049;
  }
</style>