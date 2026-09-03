# Undeepfryer

https://olivewong.github.io/live-photo-undeepfryer/

extract raw, unprocessed frames (.png) from live photo .mov

converts .mov to .png in browser via ffmpeg

## Motivation

goal: get the raw, unprocessed, undeepfried frames from live photos

problem: live photos get cropped, worse lighting, deepfried, processed as soon as you select a frame. there's no way to disable this behavior in photos. 

solution: Use ffmpeg to extract the undeepfried frames from the .mov file

this is well-documented:
- [Live Photo Quality Loss When Changing Key Frame](https://discussions.apple.com/thread/250580900?sortBy=rank)
- [Live photo zooms in when choosing another photo as Key Photo](https://discussions.apple.com/thread/250990855?sortBy=rank)
- [Apple Community thread 255189717](https://discussions.apple.com/thread/255189717?sortBy=rank)
- [how do you stop live photos from zooming in when you save a picture](https://www.reddit.com/r/ApplePhotos/comments/1s39bc3/how_do_you_stop_live_photos_from_zooming_in_when/)
- [Selecting new key photo on a Live Photo causes it to automatically zoom I.](https://discussions.apple.com/thread/255679984?sortBy=rank)
to reproduce this: open a live photo, edit, tap a different key frame, see it looks nice, then save the keyframe, observe different lighting and crop. Or, export originals as shown below and observe that the HEIC (live photo) looks very different from the .mov frames.



## Usage

On Mac: export via image capture on iPhone to mac. then export the `.mov` from **“Export Unmodified Original”**. [how to get this](https://support.apple.com/guide/photos/export-photos-videos-slideshows-and-memories-pht6e157c5f/mac), then upload it here.

then choose the exported `.MOV` here 

this does not upload anywhere. everything is done locally in your browser (s/o ffmpeg).

this runs ffmpeg in webassembly and decodes every video frame from the `.MOV` into pngs. you can save one frame or download them all as a zip.


