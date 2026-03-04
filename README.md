<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>gtnocayt's gif to roblox converter</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
    <style>
        :root { --blue: #0078d4; --dark: #111112; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--dark); color: white; display: flex; flex-direction: column; align-items: center; padding: 50px; }
        .card { background: #1f1f21; padding: 30px; border-radius: 12px; border: 1px solid #333; width: 450px; text-align: center; }
        .drop-zone { border: 2px dashed #444; padding: 40px; border-radius: 8px; cursor: pointer; margin: 20px 0; }
        #status { font-family: monospace; color: #aaa; margin: 10px 0; }
        .btn { background: var(--blue); color: white; border: none; padding: 12px 20px; border-radius: 6px; cursor: pointer; font-weight: 600; display: none; width: 100%; }
        #preview { display: grid; grid-template-columns: repeat(auto-fill, minmax(60px, 1fr)); gap: 8px; margin-top: 20px; max-height: 250px; overflow-y: auto; background: #000; padding: 10px; }
        img { width: 100%; border-radius: 4px; border: 1px solid #333; }
    </style>
</head>
<body>

<div class="card">
    <h2>gtnocayt's gif to rblx gui converter</h2>
    <div class="drop-zone" id="dz">Click to Upload GIF</div>
    <input type="file" id="fi" accept="image/gif" style="display:none">
    <div id="status">Ready.</div>
    <div id="preview"></div>
    <button id="dl" class="btn">DOWNLOAD ALL PNGs</button>
</div>

<script>
    const fi = document.getElementById('fi');
    const dz = document.getElementById('dz');
    const status = document.getElementById('status');
    const preview = document.getElementById('preview');
    const dl = document.getElementById('dl');
    let zip = new JSZip();

    dz.onclick = () => fi.click();

    fi.onchange = async (e) => {
        const file = e.target.files[0];
        if (!file) return;

        status.innerText = "Initializing...";
        preview.innerHTML = "";
        dl.style.display = "none";
        zip = new JSZip();

        try {
            const buffer = await file.arrayBuffer();
            const decoder = new ImageDecoder({ data: buffer, type: 'image/gif' });
            
            // FIX: Specifically wait for tracks to be ready
            await decoder.tracks.ready;
            const track = decoder.tracks.selectedTrack || decoder.tracks[0];
            
            if (!track) {
                throw new Error("No image tracks found in this file.");
            }

            const frameCount = track.frameCount;
            status.innerText = `Decoding ${frameCount} frames...`;

            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');

            for (let i = 0; i < frameCount; i++) {
                const result = await decoder.decode({ frameIndex: i });
                const frame = result.image;

                canvas.width = frame.displayWidth;
                canvas.height = frame.displayHeight;
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.drawImage(frame, 0, 0);

                const dataUrl = canvas.toDataURL('image/png');
                const img = document.createElement('img');
                img.src = dataUrl;
                preview.appendChild(img);

                const base64 = dataUrl.split(',')[1];
                zip.file(`${i + 1}.png`, base64, {base64: true});

                frame.close();
                status.innerText = `Progress: ${i + 1} / ${frameCount}`;
            }

            status.innerText = "Done! Frames extracted.";
            dl.style.display = "block";

        } catch (err) {
            status.innerText = "Error: " + err.message;
            console.error(err);
        }
    };

    dl.onclick = async () => {
        const blob = await zip.generateAsync({type: "blob"});
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = "frames.zip";
        link.click();
    };
</script>
</body>
</html>
