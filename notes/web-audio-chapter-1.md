# WebAudio Study Guide - Chapter 1

## I. Overview

The [WebAudio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) can be used to build a variety of audio-related applications and games, that utilize advanced music synthesis and visualizations.

- Some handy links:
  - The book we are referencing is online and free in HTML and PDF formats: https://webaudioapi.com/book/
  - Sample code from book - https://webaudioapi.com/samples/
  - Main site for book - https://webaudioapi.com
  - Web Audio Documentation:
    - https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
    - https://www.w3.org/TR/webaudio/

## II. Chapter 1 - Fundamentals

1. Which technology does the author claim was the first cross-platform way to play audio on the web?

2. The Web Audio API is built around the concept of an _______________. The audio context is a directed graph of _______________ that defines how the audio stream flows from its _______________ (often an audio file) to its _______________ (often your speakers). As audio passes through each node, its properties can be ______________. The simplest audio context is a connection directly from a ______________ to a _______________

3. Note the more complicated web audio graph in Figure 1-2. What do the terms "wet" and "dry" mean in the context of the processing of audio sound signals? (google it!)

4. *Note the cross-platform way of obtaining an `AudioContext` - in this class we'll use the following version (no answer required)*:

```js
	let audioCtx = new (window.AudioContext || window.webkitAudioContext);
```

5. Give at least 2 examples of each of the following node types:

    - Source nodes:

    - Modification nodes:

    - Analysis nodes:

    - Destination nodes:

6. *Note*:
    - *Figure 1.3 is the simplest possible audio graph that actually does something besides simply playing the sound (no answer required)*
    - *Figure 1-4. Multiple sources with individual gain control as well as a master gain (no answer required)*

7. In terms of physics, sound is a _______________ wave (sometimes called a pressure wave) that travels through air or another medium. 

8. Mathematically, sound can be represented as a _______________, which ranges over pressure values across the domain of _______________.

9. According to the author, what is the most common *sample rate* and *bit depth* for typical digital audio?

10. The process of converting analog signals into digital ones is called _______________ or _______________

11. According to the author, what is the typical sample rate for a telephone system?

12. What is the benefit (and associated trade-off) of increasing the sample rate of a sound?

13. *Pulse-code modulation* (PCM) treats sounds as _______________

14. When we load/analyze/manipulate an existing sound file using WebAudio, the loading code for a sound that is in a *lossy* or *lossless* format is the same. But it is still important to know the differences:

    - With _______________ compression the bits are identical before and after the compression process

    - With _______________ compression some bits are thrown away (the ones we hope the user won't hear anyway)

    - Give an example of a *lossless* audio file format _______________

    - Give an example of a *lossy* audio file format _______________


## III. Example Code

- Below is an expanded version of the *Putting It All Together* example from the book:
  - a live version is here: http://igm.rit.edu/~acjvks/courses/shared/330/web-audio/sg/putting-it-all-together.html
- Note that here we are loading 2 files from disk and playing them at the same time. In your Audio Visualizer projecrs, it's more likely that you will be playing just one sound (song) at a time using the `<audio>` element
- Thus there is quite a bit of file loading code here that yoou likely won't need in the future (class `BufferLoader`) - so try to concentrate instead on the `createAudioGraph()` function
- P.S. the sound files are in myCourses

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8" />
	<title></title>
</head>
<body>
<button id="startButton">Click me to hear 2 audio tracks played at the same time</button>
<button id="stopButton">Click me to stop both tracks</button>
<script>

/*
https://developer.mozilla.org/en-US/docs/Web/API/AudioContext
The AudioContext interface represents an audio-processing graph built from audio modules linked together, each represented by an AudioNode. 
An audio context controls both the creation of the nodes it contains and the execution of the audio processing, or decoding. 
You need to create an AudioContext before you do anything else, as everything happens inside a context.
*/
let audioCtx;

 // if we keep a reference to these we can .start() and .stop() them
let source1,source2;

const trackPaths = { // we'll name our sound files to make it easier to keep track of them
	'mainTrack' : './sounds/hyper-reality/br-jam-loop.wav',
	'laughTrack' : './sounds/hyper-reality/laughter.wav',
 };


startButton.onclick = init;
stopButton.onclick = stopAudio;

function init(){
  if(audioCtx){
    audioCtx.close();
  }
  audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  let bufferLoader = new BufferLoader(audioCtx,trackPaths,createAudioGraph);
  bufferLoader.loadTracks();
}

function stopAudio(){
  if (source1) source1.stop();
  if (source2) source2.stop();
}

function createAudioGraph(bufferObj) {
  // Create two sources and play them both together.
	source1 = audioCtx.createBufferSource();
  source2 = audioCtx.createBufferSource();
  
  source1.buffer = bufferObj["mainTrack"];
  source2.buffer = bufferObj["laughTrack"];

  source1.connect(audioCtx.destination);
  source2.connect(audioCtx.destination);
  
  source1.start(0);
  source2.start(0);
}



class BufferLoader{
	constructor(ctx, trackPaths, callback) {
		this.ctx = ctx;
		this.trackPaths = trackPaths; // ex. {"trackName" :"trackURL", ...}
		this.callback = callback;
		this.trackBuffers = {};				// will be populated with {"trackName" : buffer, ...}
		this.loadCount = 0;
		this.numToLoad = Object.keys(this.trackPaths).length;
	}
	
	loadTracks(){
		for (let trackName of Object.keys(this.trackPaths)){
			let trackURL = this.trackPaths[trackName];
			this.loadBuffer(trackName,trackURL);
		}
	}
	
	loadBuffer(trackName,trackURL) {
		const request = new XMLHttpRequest();
		request.responseType = "arraybuffer";
		
		// if there is an error with fetching the audio files, log it out
		request.onerror = e => console.error('BufferLoader: XHR error');

		request.onload = e => {
			// the audio file data is in request.response
			const arrayBuffer = request.response;
			
			const decodeSuccess = buffer => {
				if(buffer) {
					this.trackBuffers[trackName] = buffer;
					if (++this.loadCount == this.numToLoad){
						this.callback(this.trackBuffers);
					}
				}else{
					console.error('error decoding file data: ' + url);
					return;
				}
				
			};
				
			const decodeError = e => {
					console.error('decodeAudioData error', e);
			};
			
			this.ctx.decodeAudioData(arrayBuffer,decodeSuccess,decodeError);
		};
		
		request.open("GET", trackURL, true);
		request.send();
	} // end method loadBuffer()
	
} // end class BufferLoader
	
</script>
</body>
</html>

```


