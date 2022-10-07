# ffmpeg

## Stream information

```
ffprobe -show_streams ./xxxx.MOV
```

## Audio track dump

```
ffmpeg -report -i audio.wav -af ashowinfo -f null /dev/null
```

![](../.gitbook/assets/coding\_ffmpeg\_audio\_filter.png)

## Audio quantization and sampling

![](../.gitbook/assets/coding\_ffmpeg\_audio\_sampling.png)

## pts – Presentation timestamp

![](../.gitbook/assets/coding\_ffmpeg\_pts.png)

Examples:

Video (fps = 1000):

```
main: Demuxer gave packet of stream_index 1
main: Going to reencode&filter the frame
main: Demuxed packet stream 1 (stream  time_base = 1/1000), packet->pts 9249
main: Demuxed packet stream 1 (encoder time_base = 1/1000), packet->pts 9249 (after rescaling)
main: Decoded frame stream 1, frame->pts=9249, frame->bestefforts_ts=9249
filter_encode_write_frame: Pushing decoded frame to filters
filter_encode_write_frame: Pulling filtered frame from filters
encode_write_frame: Encoding frame
encode_write_frame: Muxing packet (encoder.time_base = {1/1000}, pts = 8289, before rescale) 
encode_write_frame: Muxing packet (stream.time_base  = {1/1000}, pts = 8289)
```

Audio (sampling rate = 48000):

```
main: Demuxed packet stream 0 (stream  time_base = 1/1000), packet->pts 9217
main: Demuxed packet stream 0 (encoder time_base = 1/1000), packet->pts 9217 (after rescaling)
main: Decoded frame stream 0, frame->pts=9217, frame->bestefforts_ts=9217
filter_encode_write_frame: Pushing decoded frame to filters
filter_encode_write_frame: Pulling filtered frame from filters
encode_write_frame: Encoding frame
encode_write_frame: Muxing packet (encoder.time_base = {1/48000}, pts = 442104, before rescale) 
encode_write_frame: Muxing packet (stream.time_base  = {1/1000}, pts = 9211)

```

In both outputs search for multiple instances of (time\_base = {1/XXX}, pts = YYY))
