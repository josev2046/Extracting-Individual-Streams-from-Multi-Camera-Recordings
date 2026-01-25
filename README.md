# Extracting individual streams from multi-camera recordings - LinkedIn Post 25th January 2026
<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/455695ac-10ff-4c7f-86ef-e33beb0fbb39" />

In my daily SA engagements with broadcasters, and archivists, I often come across hardware encoders that bundle multiple camera angles into a single container. While standard players like QuickTime would display the primary feed, the file actually serves as a digital vessel holding distinct streams for every connected camera and microphone. The challenge lies in extracting these "buried" angles for editing or review without incurring the quality loss or significant time penalties associated with re-encoding. To solve this, we can use ffmpeg to surgically map each internal stream to a distinct output file using the stream copy function (-c copy), which transfers data bit-for-bit to ensure the process is instantaneous and mathematically identical to the source.

For a practical example, consider a source file named `recording_raw.mp4` containing four video inputs but only three active audio tracks. I would use the command `ffmpeg -i "recording_raw.mp4" -map 0:v:0 -map 0:a:0 -c copy "output_camera_1.mp4" ...`, repeating the mapping for each pair while omitting the audio map for the fourth silent stream to prevent errors.

```bash
ffmpeg -i "recording_raw.mp4" \
 -map 0:v:0 -map 0:a:0 -c copy "output_camera_1.mp4" \
 -map 0:v:1 -map 0:a:1 -c copy "output_camera_2.mp4" \
 -map 0:v:2 -map 0:a:2 -c copy "output_camera_3.mp4" \
 -map 0:v:3 -c copy "output_camera_4.mp4"
```

This method is invaluable in scenarios such as judicial proceedings, university lecture captures, and even esports, where synchronised but separate feeds are required. If you run into issues, remember to wrap filenames in quotes to handle space characters and verify your stream indices with a simple ffmpeg -i filename.mp4 check to avoid mapping "ghost streams" that don't exist.

Hope this helps.
</JV>

---

# Unlocking hidden streams: extracting individual angles from multi-camera recordings

If you work in broadcasting, archiving, or AV support, you have likely encountered a specific type of video file that looks normal on the surface but behaves oddly. You play it in standard players, and you see a single video feed. However, the file size is massive, and the metadata suggests there is more going on under the bonnet.

This is common with certain hardware encoders used in professional environments. To save on file management, these devices bundle multiple camera angles and microphone inputs into a single "container" file.

The problem is that standard video players usually only recognise the primary track. The other angles—perhaps a wide shot of a courtroom or a presentation slide feed—are technically there, sitting inside the file, but they are inaccessible for editing or review.

You *could* re-encode the file to separate them, but that takes ages and degrades the picture quality. A much better solution is to use FFmpeg to extract the streams.

To understand the fix, you have to understand the file. Think of a video file not as a single photo, but as a digital rucksack (the **container**). Inside that rucksack, you can pack multiple items: distinct video reels, several audio tapes, and subtitle tracks (the **streams**).

Most players just look in the main pocket and ignore the side pockets. We need a way to reach into the side pockets and pull those extra items out into their own separate rucksacks.

We do this using a process called **stream copying**.

We use the "copy" function for this work. It tells the software: *"Don't look at the video data, don't compress it, and don't change it. Just move it from File A to File B."*

Because the computer isn't doing any heavy maths to redraw the image (transcoding), the process is both **instantaneous:** (it runs as fast as your hard drive can write data)
and **lossless:** (the output is mathematically identical to the source.)


Let’s look at a real-world scenario. You have a source file containing:

* **4 Video inputs** (Camera 1, Camera 2, Camera 3, Camera 4)
* **3 Audio inputs** (Mics for Cameras 1-3, but Camera 4 is silent)

We want to split these into four separate files so an editor can work with them easily.

<img width="845" height="655" alt="image" src="https://github.com/user-attachments/assets/1f7f1fb1-7ac2-4bd7-9ea1-0a47fb5bc628" />

We use the map flag to tell the software which specific stream goes to which output file:

```bash
ffmpeg -i "recording_raw.mp4" \
 -map 0:v:0 -map 0:a:0 -c copy "output_camera_1.mp4" \
 -map 0:v:1 -map 0:a:1 -c copy "output_camera_2.mp4" \
 -map 0:v:2 -map 0:a:2 -c copy "output_camera_3.mp4" \
 -map 0:v:3 -c copy "output_camera_4.mp4"
```


Let's break it down:

* **`-i` (Input):** This selects your source file.
* **`-map 0:v:0`:** This creates a map from Input 0, Video Track 0.
* **`-map 0:a:0`:** This maps Input 0, Audio Track 0.
* **`-c copy`:** This copies the data without re-encoding.
* **Output Name:** The destination file (e.g., `output_camera_1.mp4`).

You simply repeat this block for every angle you want to extract.

**Note on the fourth stream:** In our example, the fourth camera didn't have a corresponding microphone. If we tried to map an audio track that doesn't exist, the software would throw an error and stop. Therefore, for the fourth output line, we only map the video.

## Troubleshooting

If you are trying this and it isn't working, check these three things:

**1. Check the map first**
Before you run the extraction, run a simple check command to see what is actually inside your file. This ensures you don't try to map a "ghost stream" that doesn't exist.
`ffmpeg -i "recording_raw.mp4"`

**2. Mind the spaces**
If your filename contains spaces, the command line will get confused. Always wrap your filenames in double quotes.

**3. Silent tracks**
As mentioned in the example, ensure you aren't asking for audio tracks that aren't there. If a camera feed is silent, just map the video.
