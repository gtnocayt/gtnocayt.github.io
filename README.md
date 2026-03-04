<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>GIF Frame Master</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
    <script src="https://unpkg.com/omggif@1.0.10/omggif.js"></script>
    <style>
        :root { --accent: #00a2ff; --bg: #0f0f11; --card: #1c1c1f; }
        body { font-family: 'Segoe UI', system-ui, sans-serif; background: var(--bg); color: white; display: flex; flex-direction: column; align-items: center; padding: 40px; }
        .container { background: var(--card); border: 1px solid #333; padding: 30px; border-radius: 20px; width: 100%; max-width: 500px; text-align: center; box-shadow: 0 20px 40px rgba(0,0,0,0.4); }
        h1 { margin-top: 0; color: var(--accent); letter-spacing: 1px; }
        .drop-zone { border: 2px dashed #444; padding: 40px; border-radius: 15px; margin: 20px 0; cursor: pointer; transition: 0.3s; }
        .drop-zone:hover { border-color: var(--accent); background: rgba(0, 162, 255, 0.05); }
        #status { font-family: monospace; font-size: 13px; color: #888; margin-bottom: 15px; }
        .btn { background: var(--accent); color: white; border: none; padding: 12px 25px; border-radius: 8px; font-weight: bold; cursor: pointer; display: none; width: 100%; transition: 0.2s; }
        .btn:hover { filter: brightness(1.2); transform: translateY(-2px); }
        #preview { display: grid; grid-template-columns: repeat(auto-fill, minmax(60px, 1fr)); gap: 8px; margin-top: 20px; max-height: 250px; overflow-y: auto; background: #111; padding: 10px; border-radius: 10px; }
        .frame-thumb { width: 100%; aspect-ratio: 1; object-fit: contain; background: #222; border-radius: 4px; border: 1px solid #333; }
        progress { width: 100%; height: 10px; border-radius: 5px; accent-color: var(--accent); display: none; }
    </style>
</head>
<body>

<div class="container">
    <h1>GIF SPLITTER</h1>
    <div class="drop-zone" id="dropZone">
        <p>Click or Drop GIF here</p>
        <input type="file" id="fileInput" accept="image/gif" style="display:none">
    </div>
    
    <div id="status">Ready...</div>
    <progress id="prog" value="0" max="100"></progress>
    <div id="preview"></div>
    <button id="dlBtn" class="btn">DOWNLOAD ZIP FOR ROBLOX</button>
</div>

<script>
    const fileInput = document.getElementById('fileInput');
    const dropZone = document.getElementById('dropZone');
    const status = document.getElementById('status');
    const dlBtn = document.getElementById('dlBtn');
    const prog = document.getElementById('prog');
    const preview = document.getElementById('preview');

    let zip = new JSZip();

    dropZone.onclick = () => fileInput.click();

    fileInput.onchange = async (e) => {
        const file = e.target.files[0];
        if (!file) return;

        status.innerText = "Reading Binary Data...";
        preview.innerHTML = "";
        dlBtn.style.display = "none";
        prog.style.display = "block";
        zip = new JSZip();

        const buffer = await file.arrayBuffer();
        const reader = new Uint8Array(buffer);
        
        try {
            const gr = new giffer.GifReader(reader);
            const frameCount = gr.numFrames();
            status.innerText = `Found ${frameCount} frames. Processing...`;

            // Canvas for rendering frames
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = gr.width;
            canvas.height = gr.height;

            // Handle frame-by-frame data
            let pixelBuffer = new Uint8ClampedArray(gr.width * gr.height * 4);

            for (let i = 0; i < frameCount; i++) {
                gr.decodeAndBlitFrameRGBA(i, pixelBuffer);
                const imageData = new ImageData(pixelBuffer, gr.width, gr.height);
                ctx.putImageData(imageData, 0, 0);

                const dataUrl = canvas.toDataURL('image/png');
                
                // Add to Preview
                const img = document.createElement('img');
                img.src = dataUrl;
                img.className = "frame-thumb";
                preview.appendChild(img);

                // Add to ZIP
                const base64 = dataUrl.split(',')[1];
                zip.file(`${i + 1}.png`, base64, {base64: true});

                // Update Progress
                prog.value = ((i + 1) / frameCount) * 100;
            }

            status.innerText = `Success! ${frameCount} frames ready.`;
            dlBtn.style.display = "block";
        } catch (err) {
            console.error(err);
            status.innerText = "Error: This GIF is corrupted or too complex.";
        }
    };

    dlBtn.onclick = async () => {
        status.innerText = "Compressing ZIP...";
        const blob = await zip.generateAsync({type: "blob"});
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = "roblox_gif_frames.zip";
        link.click();
        status.innerText = "Downloaded!";
    };
</script>

</body>
</html>
