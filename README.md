# my-second-test
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Universal Boutique Loader</title>
    <style>
        body { background: #000; color: #0f0; font-family: 'Courier New', monospace; display: flex; flex-direction: column; align-items: center; padding: 20px; margin: 0; }
        .zone { border: 2px dashed #0f0; padding: 40px; margin: 10px; width: 80%; text-align: center; transition: 0.3s; }
        .zone:hover, .drag-over { background: rgba(0, 255, 0, 0.1); border-style: solid; }
        button { background: #000; color: #0f0; border: 1px solid #0f0; padding: 15px 30px; cursor: pointer; font-weight: bold; margin: 10px; }
        button:hover { background: #0f0; color: #000; }
        h2 { text-transform: uppercase; letter-spacing: 2px; }
        #file-status { font-size: 0.8em; margin-top: 10px; color: #888; }
    </style>
</head>
<body>
    <h2>1. Remote Git Execution</h2>
    <div class="zone">
        <button onclick="launchAllRemotes()">RUN ALL 5 GITHUB REPOS</button>
        <p>Redirects each repo into its own blank tab.</p>
    </div>
    <h2>2. Local File Drop Zone</h2>
    <div id="drop-zone" class="zone">
        DRAG & DROP FOLDERS OR FILES HERE
        <div id="file-status">Accepts single files or entire project directories</div>
    </div>
    <script>
        const gitRepos = [
            "https://github.com",
            "https://github.com",
            "https://github.com",
            "https://github.com",
            "https://github.com"
        ];
        // --- PART 1: REMOTE GITHUB LOADING ---
        async function launchAllRemotes() {
            for (const url of gitRepos) {
                const path = url.replace('https://github.com/', '').replace('.git', '');
                const rawBase = `https://raw.githubusercontent.com{path}/master/`;
                try {
                    const resp = await fetch(rawBase + "index.html");
                    if (!resp.ok) throw new Error();
                    openInNewTab(await resp.text(), rawBase);
                } catch {
                    // Fallback to main branch
                    const mainBase = rawBase.replace('/master/', '/main/');
                    const resp2 = await fetch(mainBase + "index.html");
                    if (resp2.ok) openInNewTab(await resp2.text(), mainBase);
                }
            }
        }
        // --- PART 2: DRAG & DROP LOCAL FILES ---
        const dropZone = document.getElementById('drop-zone');
        ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(evt => {
            dropZone.addEventListener(evt, e => { e.preventDefault(); e.stopPropagation(); });
        });
        dropZone.addEventListener('dragover', () => dropZone.classList.add('drag-over'));
        dropZone.addEventListener('dragleave', () => dropZone.classList.remove('drag-over'));
        dropZone.addEventListener('drop', async (e) => {
            dropZone.classList.remove('drag-over');
            const items = e.dataTransfer.items;
            let files = [];
            // Handle both folder drops and multiple file selections
            for (let item of items) {
                if (item.kind === 'file') {
                    const entry = item.webkitGetAsEntry();
                    if (entry) await scanFiles(entry, files);
                }
            }
            const indexFile = files.find(f => f.name.toLowerCase() === 'index.html');
            if (indexFile) {
                const reader = new FileReader();
                reader.onload = (event) => openInNewTab(event.target.result, null, files);
                reader.readAsText(indexFile);
            } else {
                alert("No index.html detected in the dropped files.");
            }
        });
        async function scanFiles(entry, fileList) {
            if (entry.isFile) {
                const file = await new Promise(res => entry.file(res));
                fileList.push(file);
            } else if (entry.isDirectory) {
                const reader = entry.createReader();
                const entries = await new Promise(res => reader.readEntries(res));
                for (let e of entries) await scanFiles(e, fileList);
            }
        }
        // --- CORE EXECUTION ENGINE ---
        function openInNewTab(htmlContent, remoteBaseUrl = null, localFiles = []) {
            const win = window.open('about:blank', '_blank');
            let finalHtml = htmlContent;
            if (remoteBaseUrl) {
                // If it's a GitHub file, use <base> to load dependencies from the cloud
                const baseTag = `<base href="${remoteBaseUrl}">`;
                finalHtml = finalHtml.includes('<head>') ? finalHtml.replace('<head>', '<head>' + baseTag) : baseTag + finalHtml;
            } else if (localFiles.length > 0) {
                // If it's local, we would typically use a Service Worker for a full folder environment.
                // For a simple 'about:blank' redirect, we inject the raw HTML.
                console.log("Local files loaded into memory.");
            }
            const blob = new Blob([finalHtml], { type: 'text/html' });
            win.location.href = URL.createObjectURL(blob);
        }
    </script>
</body>
</html>
