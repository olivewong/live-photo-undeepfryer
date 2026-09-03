# live photo rescuer

extract raw, unprocessed frames from live photo

## why?

goal: get the raw, unedited frames from live photos.

like many others, i was deeply frustrated that live photos get cropped, worse lighting, deepfried and harsh as soon as you select a frame. there's no way to disable this behavior in photos. some examples:

- [Live Photo Quality Loss When Changing Key Frame](https://discussions.apple.com/thread/250580900?sortBy=rank)
- [Live photo zooms in when choosing another photo as Key Photo](https://discussions.apple.com/thread/250990855?sortBy=rank)
- [Apple Community thread 255189717](https://discussions.apple.com/thread/255189717?sortBy=rank)
- [how do you stop live photos from zooming in when you save a picture](https://www.reddit.com/r/ApplePhotos/comments/1s39bc3/how_do_you_stop_live_photos_from_zooming_in_when/)
- [Selecting new key photo on a Live Photo causes it to automatically zoom I.](https://discussions.apple.com/thread/255679984?sortBy=rank)

if you don't know what i'm talking about, open a live photo, edit, tap a different key frame, see it looks nice, then save the keyframe - completely different lighting.

i spent some time this afternoon figuring out how to get the raw data from my iphone, to my computer, thru an ffmpeg incantation, so i thought i'd try to make it a little easier.

## use it

export the `.mov` from **“Export Unmodified Original”**. [how to get this](https://support.apple.com/guide/photos/export-photos-videos-slideshows-and-memories-pht6e157c5f/mac), then upload it here.

then choose the exported `.MOV` here and hit `extract pics`.

this does not upload anywhere. everything is done locally in your browser (s/o ffmpeg).

under the hood, the page runs ffmpeg in webassembly and decodes every video frame from the original live-photo `.MOV` into pngs. you can save one frame or download them all as a zip.

repo: https://github.com/olivewong/live-photo-undeepfryer

## implementation + context

### a live photo is not one image

conceptually, a live photo contains at least two important pieces:

- a high-resolution still image, usually HEIC on newer iphones
- a short paired video, stored as MOV

those are different captures/data streams. the HEIC is the normal high-resolution key photo. the MOV contains the surrounding live-photo frames.

when you scrub through a live photo and choose a different key frame, you are choosing a frame from the video component, not magically revealing another full-resolution HEIC that was hidden behind the first one.

### why the HEIC and MOV frames look different

photos has separate still-photo and video pipelines. the HEIC can have computational photography processing that is specific to the captured still. the paired MOV is compressed video and has its own resolution, crop, stabilization, exposure, color and tone-mapping characteristics.

so `IMG_8427.HEIC` and frame 42 of `IMG_8427.MOV` are not expected to be pixel-identical even if they represent almost the exact same instant.

also, "raw" here means **the frames decoded directly from the original paired live-photo MOV, without going through photos' make-key-photo rendering path**. it does **not** mean camera RAW / untouched sensor values. the MOV has already gone through the iphone camera/video pipeline and video compression.

### original MOV vs `IMG_E....MOV`

apple's photo library can contain multiple resources for an edited live photo.

in photokit, apple explicitly distinguishes:

- [`pairedVideo`](https://developer.apple.com/documentation/photos/phassetresourcetype/pairedvideo): the **original video data component** of a live photo
- [`fullSizePairedVideo`](https://developer.apple.com/documentation/photos/phassetresourcetype/fullsizepairedvideo): the **current rendered output** for an edited live photo

when exporting from a mac with image capture, you may see files like:

```text
IMG_8427.HEIC
IMG_8427.MOV
IMG_E8427.MOV
```

`IMG_8427.MOV` is the one this tool is primarily interested in: the original paired movie.

an `IMG_E....MOV` is associated with an edited/rendered version. it can still be interesting to inspect, and ffmpeg can extract its frames too, but it isn't the same thing as the untouched original paired video.

### why not just save a new key photo?

because that is exactly the path this project is trying to avoid.

photos shows you a frame while you're scrubbing, but after you make it the key photo and save, photos can render a different-looking result. the reports linked above describe the same crop/zoom and quality-change behavior across multiple ios generations.

instead, live photo rescuer takes the original MOV and asks ffmpeg for every video frame directly:

```text
original live-photo MOV
        -> ffmpeg
        -> frame_0001.png
        -> frame_0002.png
        -> ...
```
