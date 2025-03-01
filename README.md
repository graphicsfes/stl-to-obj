<h1>STL to OBJ converter in Javascript</h1>
<a href="https://www.freestlplus.com/p/free-stl-to-obj-converter.html">Live Preview STL to OBJ converter</a>
<h2>JS code</h2>
<pre><code>
  &lt;script&gt;
  function parseSTL(stlBuffer) {
    const text = new TextDecoder().decode(stlBuffer);
    if (text.startsWith("solid")) {
        return parseSTLText(text);
    } else {
        return parseSTLBinary(stlBuffer);
    }
}

function parseSTLText(text) {
    const lines = text.split("\n");
    const vertices = [];
    const faces = [];
    const vertexMap = new Map();
    
    function addVertex(v) {
        const key = v.join(" ");
        if (!vertexMap.has(key)) {
            vertexMap.set(key, vertices.length + 1);
            vertices.push(v);
        }
        return vertexMap.get(key);
    }

    for (let i = 0; i < lines.length; i++) {
        const parts = lines[i].trim().split(/\s+/);
        if (parts[0] === "vertex") {
            const vIndex = addVertex(parts.slice(1).map(Number));
            faces.push(vIndex);
        }
    }

    return generateOBJ(vertices, faces);
}

function parseSTLBinary(buffer) {
    const dataView = new DataView(buffer);
    const numTriangles = dataView.getUint32(80, true);
    const vertices = [];
    const faces = [];
    const vertexMap = new Map();

    function addVertex(x, y, z) {
        const key = `${x} ${y} ${z}`;
        if (!vertexMap.has(key)) {
            vertexMap.set(key, vertices.length + 1);
            vertices.push([x, y, z]);
        }
        return vertexMap.get(key);
    }

    let offset = 84;
    for (let i = 0; i < numTriangles; i++) {
        offset += 12; // Skip normal
        const v1 = addVertex(dataView.getFloat32(offset, true), dataView.getFloat32(offset + 4, true), dataView.getFloat32(offset + 8, true));
        offset += 12;
        const v2 = addVertex(dataView.getFloat32(offset, true), dataView.getFloat32(offset + 4, true), dataView.getFloat32(offset + 8, true));
        offset += 12;
        const v3 = addVertex(dataView.getFloat32(offset, true), dataView.getFloat32(offset + 4, true), dataView.getFloat32(offset + 8, true));
        offset += 14; // Skip attribute byte count
        faces.push([v1, v2, v3]);
    }

    return generateOBJ(vertices, faces.flat());
}

function generateOBJ(vertices, faces) {
    let obj = "";
    vertices.forEach(v => obj += `v ${v.join(" ")}\n`);
    for (let i = 0; i < faces.length; i += 3) {
        obj += `f ${faces[i]} ${faces[i + 1]} ${faces[i + 2]}\n`;
    }
    return obj;
}
let fileNameWithoutExt = 'output'
// File input & conversion
document.getElementById("stlInput").addEventListener("change", function(event) {
    const file = event.target.files[0];
    fileNameWithoutExt = file.name.replace(/\.[^/.]+$/, "");
    const reader = new FileReader();
    reader.onload = function() {
        const objData = parseSTL(reader.result);
        downloadOBJ(objData, fileNameWithoutExt+"-www_freestlplus_com.obj");
        document.getElementById("stlInput").style.display = 'none'
        document.getElementById("stlInput").insertAdjacentHTML('afterend', '<p id=\"message">"'+fileNameWithoutExt+"-www_freestlplus_com.obj"+'" download should be starting now</p><h1><a href=\"https://www.freestlplus.com/p/free-stl-to-obj-converter.html\">Try another file</a></h1>')
    };
    reader.readAsArrayBuffer(file);
});

// Download function
function downloadOBJ(objString, filename) {
    const blob = new Blob([objString], { type: "text/plain" });
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = filename;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}
  &lt;/script&gt;
</code></pre>
