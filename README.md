# Acoustic Calssification & Segmentation 

Simple audio segmenter to isolate speech portion out of audio streams. Uses a simple feedforward MLP for classification (implemented using `tensorflow`) and heuristic smoothing methods to increase the recall of speech segments. 

This version modified from brandeis-llc repository to use applause, speech, music, noise, and silence as possible labels, and to handle binary classification of applause (rather than speech). 

## Requirements 

* System packages: [`ffmpeg`](http://ffmpeg.org/download.html)
* Python packages: 
  * `librosa`
  * `tensorflow` or `tensorflow-gpu` `>=2.0.0`
  * `numpy`
  * `scipy`
  * `scikit-learn`
  * `ffmpeg-python`

## Training 

### Pretrained model 

We provide two [pretrained models](pretrained/). Both models are trained on 3-second clips from the [MUSAN corpus](https://www.openslr.org/17/), [HIPSTAS applause samples](https://github.com/hipstas/applause-classifier), and sound from Indiana University collections using the labels: `applause`, `speech`, `music`, `noise`, and`silence`. The models are, then, serialized using [`tensorflow::SavedModel` format](https://www.tensorflow.org/guide/keras/save_and_serialize#export_to_savedmodel). The `applause-binary-xxxxxxxx` model is trained to predict applause vs non-applause; the `non-binary-xxxxxxxx` model uses all the above labels. Because of the distribution bias in the corpus (a lot fewer noise and silence samples in the training data), we randomly upsampled minority classes.

### Training pipeline

To train your own model, invoke `run.py` with `-t` flag and pass the directory name where training data is stored. You might also want to take a look at `extract_all` function in [`feature.py`](feature.py) to change how the labels are read in, if using corpora other than the MUSAN. 

## Segmentation

To run the segmenter over audio files, invoke `run.py` with `-s` flag, and pass 1) model path (feel free to use the pretrained model if needed) and 2) the directory where audio files are stored. Currently it will process all `mp3` and `wav` files in the target directory. If you want to process other types of audio file, add to or change the `file_ext` list near the bottom of [`run.py`](run.py) files. 

If you want to use binary classification, include the `-b` flag. 
If you want to specify a minimum length of segment, use the `-T` flag and sepcify a number of milliseconds. Shorter segments will be merged with the previous one.

For example:
```
python run.py -s /path/to/pretrained/applause-binary-20210203 /path/to/audio -o /path/to/output/folder -T 1000 -b
```

The processed results are stored as JSON file in the target directory named after the audio input. The JSON includes a label and start & end times in seconds. For example:

```
[
    {
        "label": "non-applause",
        "start": 0.0,
        "end": 0.64
    },
    {
        "label": "applause",
        "start": 0.65,
        "end": 6.78
    },
    {
        "label": "non-applause",
        "start": 6.79,
        "end": 373.83
    },
    {
        "label": "applause",
        "start": 373.84,
        "end": 379.55
    }
]
```
