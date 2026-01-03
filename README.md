# Easy-one-file-ESP-HTML-Editor
Easy ESP HTML‑Editor is a lightweight, fast and fully offline HTML editor that runs directly on your ESP8266 or ESP32. 
It allows you to edit the INDEX.HTML stored in LittleFS/SPIFFS directly from your browser – 
without reflashing, without external tools, modify your file, hit SAVE, and you're done.

This project is intentionally simple:
One HTML file. Zero external JavaScript. Zero dependencies. 299 KB total.


✔ Works on ESP8266 and ESP32
✔ Only one HTML file (editor.html)
✔ No external JavaScript, no frameworks, no libraries
✔ Fully offline – runs entirely on the ESP
✔ Edit any file in LittleFS/SPIFFS
✔ Ultra‑low RAM usage
✔ Zero heap fragmentation
✔ Zero CPU load
✔ Perfect for dashboards, UIs, sensor pages, config files
✔ Drop‑in module for any ESP project

Why This Editor Exists? 
Most ESP editors are:
-too heavy
-too complex
-require multiple JS files
-break on older browsers or need special upload tools
-This project solves all of that with a single file.
-299 KB. One file. Works everywhere.
//--------------------------------------------------------
```
📂 File Structure
Code
/data
 ├── editor.html
 ├── index.html
 └── any other files 
 ```
Upload the /data folder using your preferred LittleFS/SPIFFS uploader.

**********************************************************
// ----------- editor.html (One‑File Editor) ----------
*************************************************************
```
html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Easy ESP HTML Editor</title>

<style>
body { background:#222; color:#eee; font-family:Arial; }
textarea {
    width:100%; height:75vh;
    background:#111; color:#0f0;
    font-family:monospace;
}
button {
    background-color:#4CAF50;
    border:none;
    color:white;
    padding:15px 32px;
    text-align:center;
    font-size:20px;
    margin:20px;
    cursor:pointer;
    display:block;
    width:100%;
}
button:hover { background-color:#45a049; }
</style>

</head>
<body>

<h3>Easy ESP HTML Editor – Version 1.7</h3>

<textarea id="editor"></textarea>

<button onclick="saveFile()">SAVE</button>

<script>
function loadFile() {
    var xhr = new XMLHttpRequest();
    xhr.open("GET", "/load?file=index.html", true);
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4 && xhr.status == 200) {
            document.getElementById("editor").value = xhr.responseText;
        }
    };
    xhr.send();
}

function saveFile() {
    var content = document.getElementById("editor").value;

    var xhr = new XMLHttpRequest();
    xhr.open("POST", "/save?file=index.html", true);
    xhr.setRequestHeader("Content-Type", "application/octet-stream");

    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) {
            alert("index.html saved!");
        }
    };

    xhr.send(content);
}

loadFile();
</script>

</body>
</html>
```
****************************************************************
🛠 ESP Code (Endpoints)
Add these routes to your ESPAsyncWebServer:
// -------------------------------------------------------------
// Easy ESP HTML Editor – Server Endpoints
// -------------------------------------------------------------

// Serve the editor page ------------------------
```
server.on("/edit", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(LittleFS, "/editor.html", "text/html");
});
```

// Load a file from LittleFS ---------------------------
```
server.on("/load", HTTP_GET, [](AsyncWebServerRequest *request){
    if (!request->hasParam("file")) {
        request->send(400, "text/plain", "file missing");
        return;
    }
    
    String filename = "/" + request->getParam("file")->value();
    File f = LittleFS.open(filename, "r");

    if (!f) {
        request->send(404, "text/plain", "not found");
        return;
    }

    String content = f.readString();
    f.close();

    request->send(200, "text/plain", content);
});
```
// Save HTML to LittleFS --------------------------------------

```
server.on("/save", HTTP_POST,
    [](AsyncWebServerRequest *request){
        Serial.println("POST request received...");
    },
    NULL,
    [](AsyncWebServerRequest *request, uint8_t *data, size_t len, size_t index, size_t total){

        String filename = "/index.html";
        if (request->hasParam("file", true)) {
            filename = "/" + request->getParam("file", true)->value();
        }

        if (index == 0) {
            request->_tempFile = LittleFS.open(filename, "w");
        }

        if (request->_tempFile) {
            request->_tempFile.write(data, len);
        }

        if (index + len == total) {
            if (request->_tempFile) {
                request->_tempFile.close();
                request->send(200, "text/plain", "file saved");
            }
        }
    }
);

```
thats all , enjoy 
