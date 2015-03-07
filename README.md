# dilla
schedule looped playback of Web Audio events at 96 ticks per beat

Based on [ditty](https://github.com/mmckegg/ditty) and [bopper](https://github.com/mmckegg/bopper).

## Install

```
$ npm install --save dilla
```

## Usage

```
var Dilla = require('dilla');
var audioContext = new AudioContext();
var dilla = new Dilla(audioContext, options);
```

#### Options and defaults

```
{
  'tempo': 120,
  'beatsPerBar': 4,
  'loopLength': 2 // bars
}
```

### API

#### dilla.start()
#### dilla.pause()
#### dilla.stop()

#### dilla.set(id, events)
#### dilla.get(id)
#### dilla.getIds()
#### dilla.clear(id)
#### dilla.clear()

#### dilla.position
#### dilla.setPosition(position)
#### dilla.setTempo(tempo)
#### dilla.setBeatsPerBar(beats)
#### dilla.setLoopLength(bars)

### Events

#### tick

```
dilla.on('tick', function (data) {
  //fires as close to when the tick is actually updated, as possible
  data.bar, data.beat, data.tick
  data.position:string
});
```

#### step

```
dilla.on('step', function (data) {
  // piped through scheduler, fired with scheduled events
  data.event (start/stop), data.time, data.length, data.args
});
```

### Example: metronome

The "hello world" of audio libraries is usually a simple metronome, so here's a full example. This example can be run by forking this repo and running [make example](https://github.com/adamrenklint/dilla/blob/master/example.js).

```
var duration = 15;
var oscillator, gainNode;

dilla.set('metronome', [
  ['1.1.01', duration, 440],
  ['1.2.01', duration, 330],
  ['1.3.01', duration, 330],
  ['1.4.01', duration, 330],
  ['2.1.01', duration, 440],
  ['2.2.01', duration, 330],
  ['2.3.01', duration, 330],
  ['2.4.01', duration, 330]
]);

dilla.on('step', function (step) {
  if (step.event === 'start') {
    oscillator = step.context.createOscillator();
    gainNode = step.context.createGain();
    oscillator.connect(gainNode);
    gainNode.connect(step.context.destination);
    oscillator.frequency.value = step.args[2];
    gainNode.gain.setValueAtTime(1, step.time);
    oscillator.start(step.time);
  }
  else if (step.event === 'stop' && oscillator) {
    gainNode.gain.linearRampToValueAtTime(0, step.time);
    oscillator = null;
    gainNode = null;
  }
});

dilla.start();
```