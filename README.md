# Extracting individual streams from multi-camera recordings - LinkedIn Post 25th January 2026
<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/455695ac-10ff-4c7f-86ef-e33beb0fbb39" />

In my daily SA engagements with broadcasters, and archivists, I often come across hardware encoders that bundle multiple camera angles into a single container. While standard players like QuickTime would display the primary feed, the file actually serves as a digital vessel holding distinct streams for every connected camera and microphone. The challenge lies in extracting these "buried" angles for editing or review without incurring the quality loss or significant time penalties associated with re-encoding. To solve this, we can use ffmpeg to surgically map each internal stream to a distinct output file using the stream copy function (-c copy), which transfers data bit-for-bit to ensure the process is instantaneous and mathematically identical to the source.

For a practical example, consider a source file named recording_raw.mp4 containing four video inputs but only three active audio tracks. I would use the command ffmpeg -i "recording_raw.mp4" -map 0:v:0 -map 0:a:0 -c copy "output_camera_1.mp4" ..., repeating the mapping for each pair while omitting the audio map for the fourth silent stream to prevent errors.

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
