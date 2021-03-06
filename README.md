# Text-to-speech in (partially) C++ using Tacotron model + Tensorflow

Running Tacotron model in TensorFlow C++ API.

Good for running TTS in mobile or embedded device.

Code is based on keithito's tacotron implementation: https://github.com/keithito/tacotron 

## Status

Experimental.

Currently only running Tacontron model is coded in C++.

Python preprocessing is required to generate sequence data from text.
Python postprocessing is required to do `inv_preemphasis` for Tacotron generated .wav.

## Requirment

* TensorFlow r1.8+
* Ubuntu 16.04
* C++ compiler + cmake

## Dump graph.

In keithito's tacotron repo, append `tf.train.write_graph` to `Synthesizer::load` to save TensorFlow graph.


```
class Synthesizer:
  def load(self, checkpoint_path, model_name='tacotron'):

    ...

    # write graph
    tf.train.write_graph(self.session.graph.as_graph_def(), "models/", "graph.pb")
```

## Freeze graph

Freeze graph for example:

```
freeze_graph \
        --input_graph=models/graph.pb \
        --input_checkpoint=./tacotron-20180906/model.ckpt \
        --output_graph=models/tacotron_frozen.pb \
        --output_node_names=model/griffinlim/Squeeze
```

Example freeze graph file is included in this repo.

## Build

Edit tensorflow path(Assume you build TensorFlow from source code) in `bootstrap.sh`, then

```
$ ./bootstrap.sh
$ build
$ make
```

## Run

Prepare sequence JSON file.
Sequence can be generated by using `text_to_sequence()` function in keithito's tacotron repo.

See `sample/sequence01.json` for generated example.

Then,

```
$ ./tts -i ../sample/sequence01.json -g ../tacotron_frozen.pb -o output.wav
```

We need to further process generated `output.wav`.
In keithito's tacotron repo, run python script like this

```
import io
import numpy as np
from librosa import effects
from util import audio

wav = audio.load_wav("output.wav")
wav = audio.inv_preemphasis(wav)
wav = wav[:audio.find_endpoint(wav)]
out = io.BytesIO()
audio.save_wav(wav, "processed.wav")
```

example output01.wav and processed01.wav is included in `sample/`

## Performance

Currently TensorFlow C++ code path only uses single CPU core, so its slow.
Time for synthesis is roughly 10x slower on 2018's CPU than synthesized audio length(e.g. 60 secs for 6 secs audio).

## TODO

* Write all TTS pipeline fully in C++
  * [ ] Text to sequence
  * [ ] inv_preemphasis

## License

MIT license.

Pretrained model used for freezing graph is obtained from keithito's repo.

### Third party licenses

- json.hpp : MIT license
- cxxopts.hpp : MIT license
- dr_wav : Public domain
