# RYSKaframe
This is an attempt to integrate RYSK libraries with a-frame. Since A-Frame doesn't use node.js package architecture
in the usual way and reliaes on global variable ``AFRAME``, this package had to be composed in the similar manner.

If you intend to use the package together with other node.js packages and install it either through yarn or npm, the
package depends on ``@mantisvision/ryskstream``, ``@mantisvision/ryskurl`` and ``aframe`` packages. Bear in mind that 
different versions of A-Frame library don't work very well together, so check if the version required by ``@mantisvision/ryskaframe``
is the same as the one you are using, otherwise you may end up with two different versions installed. Even if it comes
to this, ``@mantisvision/ryskaframe`` doesn't actually import ``aframe`` library inside its source; instead it expects 
that you'll be the one doing the import, so you can ensure you use the right (i.e. your own) version.

Alternatively you can use the minified version of ``@mantisvision/ryskaframe`` which is bundled in the same package
and containsall its ``@mantisvision/rysk*`` dependencies withing itself.

## Install
You can install this package using one of the following commands for either yarn or npm
```
yarn add @mantisvision/ryskthreejs
npm install @mantisvision/ryskaframe
```
You can also just download the package from its repository, decompress and use only minified version ``MantisRYSKaframe.min.js``.

## Usage
You can either import the library like this:
```javascript
import "aframe";
import "@mantisvision/ryskaframe";
```
A-Frame doesn't need to be imported in the same file as ``@mantisvision/ryskaframe``, but it needs to be imported prior
to ``@mantisvision/ryskaframe``, otherwise an exception shall be thrown.

Minified version can be loaded via HTML ``<script>`` tag in the header:
```html
<script src="./scripts/aframe.min.js"></script>
<script src="./scripts/MantisRYSKaframe.min.js"></script>
```
It has no dependencies (everything is bundled inside), but again, aframe must be loaded prior to it.

``@mantisvision/ryskaframe`` registers two new components within A-Frame: ``ryskurl`` and ``ryskstream``.
For convinience, it also registers two coresponding primitives: ``<mantis-ryskurl></mantis-ryskurl>`` 
and ``<mantis-ryskstream></mantis-ryskstream>``.

### ryskurl
This component is used to create a 3D animated mesh from pre-recorded video and SYK/RYSK volumetric data:
```html
<a-scene>
	<a-entity 
		position="0 0 -2" 
		ryskurl="loop:false; data: https://www.mvkb.cc/lib/exe/fetch.php/pub/genady5.syk; video: https://www.mvkb.cc/lib/exe/fetch.php/pub/genady5.mp4" >
	</a-entity>
</a-scene>
```

#### Schema
ryskurl component is based on the following schema:
```javascript
{
	video: { type: "string", default: '' },
	data: { type: "string", default: '' },
	buffer: { type: "int", default: 50 },
	loop: { type: "boolean", default: true },
	volume: { type: "number", default: 0 }
}
```
- video: url of the video for the texture of the mesh
- data: url of the SYK/RYSK volumetric data
- buffer: size of data buffer 
- loop: marks if the video should loop after it ends
- volume: volume level from 0 (mute) to 1 (full volume)

#### Event listeners
ryskurl listens for the following events that you can emit on its element (i.e. ``<a-entity>`` or 
``<mantis-ryskurl></mantis-ryskurl>``):

- pause: pauses the video
- play: resumes playing

#### Emitted events
ryskurl emits the following events on its element (i.e. ``<a-entity>`` or  ``<mantis-ryskurl></mantis-ryskurl>``). None
of the events bubbles, so you have to attach your listeners directly to the element.

- buffering: video or data is buffering
- buffered: video and data are buffered
- bufferingData: RYSK/SYK data is being buffered
- dataBuffered: RYSK/SYK data is being buffered
- dataDecoded: one frame of SYK/RYSK data is decoded; the payload of the event contains the decoded data:
	- frameNo: sequence number of the decoded frame
	- uvs: typed array of uvs
	- indices: typed array of indices
	- vertices: typed of vertices
- waiting: video is buffering
- playing: video started/resumed playing
- ended: video has finished playing

#### mantis-ryskurl primitive
For your convinience, there is mantis-ryskurl primitive:
```html
<mantis-ryskurl 
	position="0 0 -2" 
	loop="false" 
	dataurl="https://www.mvkb.cc/lib/exe/fetch.php/pub/genady5.syk" 
	videourl="https://www.mvkb.cc/lib/exe/fetch.php/pub/genady5.mp4" >
</mantis-ryskurl>
```
Compenent properties are mapped to the HTML attributes in the following way:
- video: videourl
- data: dataurl
- buffer: buffer
- loop: loop
- volume: volume

### ryskstream
This component creates a 3D mesh from media stream and SYK/RYSK volumetric data which must be periodically delivered
through the emitting of an event on the HTML tag which bears the component.
```html
<a-scene>
	<a-entity 
		position="0 0 -2" 
		ryskstream="videoelem: videosource" >
	</a-entity>
</a-scene>
```

#### Schema
ryskstream component is based on the following schema:
```javascript
{
	videoelem: { type: "string", default: '' },
	width: { type: "int", default: 0 },
	height: { type: "int", default: 0 },
	volume: { type: "number", default: 0 }
}
```
- videoelem: id of the videoelement which will serve as the source of the media stream. Alternatively, media stream can be passed directly through the newdata event (see below)
- width: width resolution of the video; if not given, it either read from videoelem or from the data that came throuh the data event
- height: height resolution of the video; if not given, it either read from videoelem or from the data that came throuh the data event
- volume: volume level from 0 (mute) to 1 (full volume)

#### Event listeners
ryskstream listens for the following events that you can emit on its element (i.e. ``<a-entity>`` or 
``<mantis-ryskstream></mantis-ryskstream>``):
- newdata: this event should emitted each a new data has arrived. The following object should be passed as the payload:
	- version: version of the RYSK/SYK data 
	- data: Typed array containing encoded RYSK/SYK data
- newstream: instead of giving id of the video element through the HTML attribute, you can emit this event and as the payload pass directly the mediastream in the following object:
	- width: width resolution of the video passed in the stream
	- height: height resolution of the video passed in the stream
	- stream: MediaStream object

#### Emitted events
ryskstream emits the same events as ryskurl (see above)

#### mantis-ryskstream primitive
For your convinience, there is mantis-ryskstream primitive:
```html
<mantis-ryskstream 
	position="0 0 -2" 
	videoelem="videosource" >
</mantis-ryskstream>
```
Compenent properties are mapped to the HTML attributes in the following way:
- videoelem: videoelem
- width: width
- height: height
- volume: volume
