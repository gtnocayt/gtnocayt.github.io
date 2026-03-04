<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Schöne GIF Converter V2</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/libgif-js/0.0.3/libgif.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background: #121212; color: white; text-align: center; padding: 50px; }
        .box { background: #1e1e1e; border: 2px dashed #444; padding: 40px; border-radius: 20px; max-width: 500px; margin: auto; }
        .btn { background: #00a2ff; color: white; padding: 15px 30px; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; margin-top: 20px; }
        .btn:hover { background: #0077cc; }
        #status { margin-top: 20px; color: #00ffcc; font-family: monospace; }
        canvas { display: none; } /* We hide the working canvas */
        #preview { margin-top: 20px; display: flex; flex-wrap: wrap; justify-content: center; gap: 5px; max-height: 200px; overflow-y: auto; }
        .thumb { width: 50px; height: 50px; border: 1px solid #555; object-fit: contain; }
    </style>
</head>
<body>

<div class="box">
    <h1>GIF FRAME SPLITTER</h1>
    <input type="file" id="fileInput" accept="image/gif" style="display:none">
    <button class="btn" onclick="document.getElementById('fileInput').click()">SELECT GIF FILE</button>
    <div id="status">Waiting for file...</div>
    <div id="preview"></div>
    <button id="dl" class="btn" style="display:none; background:#28a745">DOWNLOAD ZIP</button>
</div>

<img id="gifTarget" style="display:none">

<script>
    const fileInput = document.getElementById('fileInput');
    const status = document.getElementById('status');
    const preview = document.getElementById('preview');
    const dlBtn = document.getElementById('dl');
    let zip = new JSZip();

    fileInput.onchange = function(e) {
        const file = e.target.files[0];
        if (!file) return;

        status.innerText = "Loading GIF...";
        preview.innerHTML = "";
        dlBtn.style.display = "none";
        zip = new JSZip();

        const img = document.getElementById('gifTarget');
        const reader = new FileReader();
        
        reader.onload = function(event) {
            img.src = event.target.result;
            
            // Wait for image to load, then use SuperGif to parse
            img.onload = function() {
                const rub = new SuperGif({ gif: img });
                rub.load(function() {
                    status.innerText = `Extracting ${rub.get_length()} frames...`;
                    
                    for (let i = 0; i < rub.get_length(); i++) {
                        rub.move_to(i);
                        const canvas = rub.get_canvas();
                        
                        // Create visible thumbnail
                        const thumb = document.createElement('img');
                        thumb.src = canvas.toDataURL();
                        thumb.className = "thumb";
                        preview.appendChild(thumb);

                        // Add to ZIP
                        const data = canvas.toDataURL('image/png').replace(/^data:image\/(png|jpg);base64,/, "");
                        zip.file((i + 1) + ".png", data, {base64: true});
                    }

                    status.innerText = "DONE! Found " + rub.get_length() + " frames.";
                    dlBtn.style.display = "inline-block";
                });
            };
        };
        reader.readAsDataURL(file);
    };

    dlBtn.onclick = function() {
        zip.generateAsync({type:"blob"}).then(function(content) {
            const a = document.createElement("a");
            a.href = URL.createObjectURL(content);
            a.download = "frames.zip";
            a.click();
        });
    };
</script>
</body>
</html>
