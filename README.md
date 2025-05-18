Here’s a detailed, step-by-step guide to legally record live TV in the USA for personal use, automate ad-skipping, and save recordings to a Jellyfin server. This assumes you're using over-the-air (OTA) broadcasts or unencrypted cable signals.

---

### **Legal Disclaimer**
- You may record live TV for personal use under U.S. copyright law (fair use/time-shifting). 
- Do **not** distribute recordings or share your Jellyfin server publicly.
- This guide is for educational purposes only.

---

### **Tools Needed**
1. **TV Tuner Hardware**: 
   - HDHomeRun (network tuner, e.g., [HDHomeRun Flex 4K](https://www.silicondust.com/)) for OTA/cable.
   - Antenna or cable connection.
2. **Software**:
   - Jellyfin (media server)
   - Comskip (ad detection)
   - FFmpeg (video processing)
   - `inotify-tools` (Linux) or `FileWatcher` (Windows) for automation (optional).

---

### **Step 1: Set Up the TV Tuner**
1. Connect your antenna/cable to the HDHomeRun.
2. Connect the HDHomeRun to your network (Ethernet/Wi-Fi).
3. Test the signal using HDHomeRun’s setup tool or apps like **Channels DVR**.

---

### **Step 2: Install and Configure Jellyfin**
1. **Install Jellyfin**:
   - Follow [official instructions](https://jellyfin.org/downloads) for your OS.
2. **Configure Live TV/DVR**:
   - Go to **Dashboard > Live TV**.
   - Under "TV Tuners," select **HDHomeRun** and enter your device’s IP.
   - Complete the channel scan to detect available channels.
3. **Schedule Recordings**:
   - Use the **Guide** to browse shows and click "Record" or set manual rules under **Library > Recordings**.

---

### **Step 3: Install Comskip for Ad Detection**
1. **Download Comskip**:
   - Windows/Linux/macOS: [Comskip GitHub](https://github.com/erikkaashoek/Comskip).
2. **Configure Comskip**:
   - Edit `comskip.ini` to match your recordings (example settings [here](https://github.com/erikkaashoek/Comskip/blob/master/comskip.ini)).
   - Test with a sample recording:
     ```
     comskip --ini=comskip.ini "your_recording.ts"
     ```
   - Comskip will generate a `.txt` or `.edl` file marking ad timestamps.

---

### **Step 4: Automate Ad Removal with FFmpeg**
1. **Install FFmpeg**:
   - [Download FFmpeg](https://ffmpeg.org/download.html) and add it to your system PATH.
2. **Create a Script to Remove Ads**:
   - Use the `.edl` file from Comskip to cut ads:
     ```bash
     ffmpeg -i "input.ts" -vf "mpdecimate,setpts=N/FRAME_RATE/TB" -af "atempo=1.0" -c:v libx264 -c:a aac -strict experimental -map_metadata 0 -f mp4 "output.mp4"
     ```
   - For precise cuts, use `-filter_complex` with the `edl` file (advanced).

---

### **Step 5: Automate the Workflow**
1. **Set Up a Post-Recording Script**:
   - Use Jellyfin’s **Webhooks** or a folder watcher to trigger scripts when new files arrive.
   - Example (Linux with `inotify`):
     ```bash
     inotifywait -m -e close_write --format "%f" /path/to/recordings | while read FILE
     do
       comskip --ini=comskip.ini "$FILE"
       ffmpeg -i "$FILE" ... (see Step 4)
       mv "processed_$FILE" /jellyfin/library/
     done
     ```
2. **Windows Alternative**:
   - Use [FileWatcher](https://www.filewatcher.com/) to run a batch/PowerShell script.

---

### **Step 6: Add Processed Files to Jellyfin**
1. Create a Jellyfin library pointing to the folder with ad-free recordings.
2. Ensure metadata (e.g., titles, thumbnails) is scraped correctly.

---

### **Troubleshooting**
- **Comskip Accuracy**: Adjust `comskip.ini` settings for your channels (e.g., `detect_method=43` for black frame detection).
- **Hardware Requirements**: Use a CPU/GPU capable of handling FFmpeg transcoding.
- **Legal Compliance**: Never bypass DRM (e.g., encrypted cable channels).

---

### **Final Notes**
- This setup requires technical skill. Test each step individually.
- For easier ad-skipping, consider **Channels DVR** (paid) or **MythTV** (open-source).
- Always respect copyright laws and keep your server private.
