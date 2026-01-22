# DRAFT #

# Extracting individual streams from multi-camera recordings

In our daily SA engagemwnts with broadcasters, archivists, and lecture capture, we frequently encounter hardware encoders—like the Epiphan Pearl series—that bundle multiple camera angles into a single `.mp4` or `.mov` container. While standard players like QuickTime often only display the primary feed, the file actually serves as a digital vessel holding distinct streams for every connected camera and microphone. The challenge lies in extracting these "buried" angles for editing or review without incurring the quality loss or significant time penalties associated with re-encoding. To solve this, we can use `ffmpeg` to surgically map each internal stream to a distinct output file using the stream copy function (`-c copy`), which transfers data bit-for-bit to ensure the process is instantaneous and mathematically identical to the source.

For a practical example, consider a source file named `recording_raw.mp4` containing four video inputs but only three active audio tracks. I would use the command `ffmpeg -i "recording_raw.mp4" -map 0:v:0 -map 0:a:0 -c copy "output_camera_1.mp4" ...`, repeating the mapping for each pair while omitting the audio map for the fourth silent stream to prevent errors. 

```bash
ffmpeg -i "recording_raw.mp4" \
  -map 0:v:0 -map 0:a:0 -c copy "output_camera_1.mp4" \
  -map 0:v:1 -map 0:a:1 -c copy "output_camera_2.mp4" \
  -map 0:v:2 -map 0:a:2 -c copy "output_camera_3.mp4" \
  -map 0:v:3 -c copy "output_camera_4.mp4"
```

This method is invaluable in scenarios such as judicial proceedings, university lecture captures, and esports, where synchronised but separate feeds are required. If you run into issues, remember to wrap filenames in quotes to handle space characters and verify your stream indices with a simple `ffmpeg -i filename.mp4` check to avoid mapping "ghost streams" that don't exist.

Hope this helps.

# Extracting Individual Streams from Multi-Camera Recordings


In my work with broadcasters, legal archiving, and lecture capture, I often encounter hardware encoders (such as the Epiphan Pearl series) that bundle multiple camera angles into a single `.mp4` or `.mov` file.
In this guide, I’ll walk you through the methodology I use for extracting separate video and audio feeds from a single multi-stream container file.

To the untrained eye—or a standard media player like QuickTime—this looks like a standard video file displaying only the primary camera feed. However, beneath the surface, the file acts as a digital container holding distinct streams for every camera and microphone connected to the device.

The challenge is extracting these "buried" angles into standalone files for editing or review, without suffering the quality loss or time penalty of re-encoding.

I leverage `ffmpeg` to surgically map each internal stream to a distinct output file. Crucially, I use the stream copy function, which copies the video data bit-for-bit rather than re-compressing it. This ensures the process is instantaneous and mathematically identical to the source.


In this example, let's assume we have a source file named `recording_raw.mp4` which contains four video inputs but only three active audio tracks.

```bash
ffmpeg -i "recording_raw.mp4" \
  -map 0:v:0 -map 0:a:0 -c copy "output_camera_1.mp4" \
  -map 0:v:1 -map 0:a:1 -c copy "output_camera_2.mp4" \
  -map 0:v:2 -map 0:a:2 -c copy "output_camera_3.mp4" \
  -map 0:v:3 -c copy "output_camera_4.mp4"
```

Let's break that down:

* `-i "filename.mp4"`: The input file. **Note:** I always enclose filenames in quotes if they contain spaces to prevent parsing errors.
* `-map 0:v:0`: Selects the **first** video track from the **first** input file.
* `-map 0:a:0`: Selects the **first** audio track from the **first** input file.
* `-c copy`: The codec is set to "copy". This skips the encoding process entirely, resulting in near-instant extraction.
* `output_name.mp4`: The destination file for that specific map pair.

**Note on Stream Mismatches:**
In the command above, you will notice the fourth output (`output_camera_4.mp4`) does not possess a mapped audio track (`-map 0:a:3`). This is deliberate. Hardware recorders often capture video on all inputs but may not have active microphones connected to every channel. If I were to attempt mapping a non-existent audio stream, `ffmpeg` would abort the entire operation.

## Use Cases

Where might you encounter this specific architecture?

1.  **Judicial Proceedings:** Modern courtrooms often record the judge, witness, defendant, and evidence presentation simultaneously. We need these separated for transcription or public record requests.
2.  **University Lecture Capture:** A lecture hall recording often captures the professor (Camera A) and the slide deck/projector feed (Camera B) into a single container.
3.  **Esports & Gaming:** High-end capture setups may record the gameplay feed and the player's "face-cam" as separate tracks within one file to simplify synchronisation during post-production.
4.  **Security Surveillance:** 360-degree cameras or multi-sensor units often output a single file containing distinct quadrants that require separation for forensic analysis.

## Troubleshooting

If your command fails, it is usually due to one of two reasons:

1.  **The Space Character:** The command line interprets a space as the end of a filename. Ensure your input is `"wrapped in quotes"` or escaped with a backslash `\`.
2.  **The Ghost Stream:** You cannot map what does not exist. Use `ffmpeg -i filename.mp4` simply to inspect the file metadata first. If the output lists `Stream #0:5` as your final stream, do not attempt to map index `6` or higher.

Hope this helps.
</JV>
