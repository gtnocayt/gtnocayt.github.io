<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>gtnocayt's GIF to rblx converter v5.3 hotfix</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
    <style>
        :root { --blue: #0078d4; --dark: #111112; --red: #ff4444; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--dark); color: white; display: flex; flex-direction: column; align-items: center; padding: 20px; }
        .card { background: #1f1f21; padding: 30px; border-radius: 12px; border: 1px solid #333; width: 90%; max-width: 800px; text-align: center; }
        .drop-zone { border: 2px dashed #444; padding: 30px; border-radius: 8px; cursor: pointer; margin: 20px 0; }
        #status { font-family: monospace; color: #00ff00; margin: 10px 0; font-size: 14px; }
        .btn-group { display: flex; gap: 10px; margin-top: 15px; }
        .btn { flex: 1; padding: 15px; border-radius: 6px; cursor: pointer; font-weight: 800; border: none; display: none; text-transform: uppercase; }
        #dl { background: var(--blue); color: white; }
        #reset { background: #333; color: #aaa; }
        
        /* Grid optimized for performance */
        #preview { 
            display: grid; grid-template-columns: repeat(auto-fill, minmax(80px, 1fr)); 
            gap: 10px; margin-top: 20px; max-height: 50vh; overflow-y: auto; 
            background: #000; padding: 15px; border-radius: 8px;
            content-visibility: auto; /* Browser optimization for long lists */
        }
        
        .frame-item { position: relative; cursor: pointer; aspect-ratio: 1; }
        .frame-item img { width: 100%; height: 100%; object-fit: cover; border-radius: 4px; border: 1px solid #333; }
        .frame-idx { position: absolute; top: 2px; left: 2px; background: rgba(0,0,0,0.7); font-size: 9px; padding: 1px 4px; pointer-events: none; }
        .frame-item:hover img { border-color: var(--red); opacity: 0.5; }
        .frame-item:hover::after { content: '❌'; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 20px; }
    </style>
</head>
<body>

<div class="card">
    <h2>gtnocayt's GIF to rblx converter v5.3 hotfix</h2>
    <div class="drop-zone" id="dz">Drop GIF here or Click to Upload</div>
    <input type="file" id="fi" accept="image/gif" style="display:none">
    <div id="status">Ready for upload...</div>
    <div id="preview"></div>
    <div class="btn-group">
        <button id="reset" class="btn">Clear All</button>
        <button id="dl" class="btn">Download Frames (ZIP)</button>
    </div>
</div>

<script>
    const fi = document.getElementById('fi');
    const dz = document.getElementById('dz');
    const status = document.getElementById('status');
    const preview = document.getElementById('preview');
    const dl = document.getElementById('dl');
    const reset = document.getElementById('reset');
    
    let frames = []; // Stores objects: { blob, id }

    dz.onclick = () => fi.click();
    reset.onclick = () => { frames = []; preview.innerHTML = ''; status.innerText = 'Cleared.'; dl.style.display = 'none'; reset.style.display = 'none'; };

    fi.onchange = async (e) => {
        const file = e.target.files[0];
        if (!file) return;

        status.innerText = "Extracting frames... this may take a second...";
        frames = [];
        preview.innerHTML = '';

        try {
            const buffer = await file.arrayBuffer();
            const decoder = new ImageDecoder({ data: buffer, type: 'image/gif' });
            await decoder.tracks.ready;
            const track = decoder.tracks.selectedTrack || decoder.tracks[0];
            
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');

            for (let i = 0; i < track.frameCount; i++) {
                const result = await decoder.decode({ frameIndex: i });
                canvas.width = result.image.displayWidth;
                canvas.height = result.image.displayHeight;
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.drawImage(result.image, 0, 0);
                
                // PERFORMANCE FIX: Use toBlob instead of toDataURL (much faster, uses less RAM)
                const blob = await new Promise(res => canvas.toBlob(res, 'image/png'));
                const url = URL.createObjectURL(blob);
                
                frames.push({ blob, url });
                
                // Add to UI immediately
                const item = document.createElement('div');
                item.className = 'frame-item';
                item.innerHTML = `<img src="${url}"><div class="frame-idx">${i+1}</div>`;
                item.onclick = () => {
                    item.style.display = 'none'; // Instant UI hide
                    frames[i] = null; // Mark for deletion
                    updateCount();
                };
                preview.appendChild(item);

                result.image.close();
                if(i % 10 === 0) status.innerText = `Loading: ${i}/${track.frameCount}`;
            }

            status.innerText = `Loaded ${track.frameCount} frames. Click any to remove!`;
            dl.style.display = "block";
            reset.style.display = "block";
        } catch (err) { status.innerText = "Error: " + err.message; }
    };

    function updateCount() {
        const remaining = frames.filter(f => f !== null).length;
        status.innerText = `${remaining} frames ready for export.`;
    }

    dl.onclick = async () => {
        status.innerText = "Building Secure ZIP...";
        const zip = new JSZip();
        const activeFrames = frames.filter(f => f !== null);
        
        // Anti-Flagging: Put frames in a dedicated folder inside the zip
        const folder = zip.folder("animation_frames");
        
        activeFrames.forEach((frame, index) => {
            folder.file(`frame_${index + 1}.png`, frame.blob);
        });

        const content = await zip.generateAsync({type: "blob"});
        const link = document.createElement('a');
        link.href = URL.createObjectURL(content);
        link.download = "Cleaned_Animation.zip";
        link.click();
        status.innerText = "Done! Open the ZIP and extract the 'animation_frames' folder.";
    };
</script>
</body>
</html>
