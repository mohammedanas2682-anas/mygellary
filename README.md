<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ultimate Secure Gallery</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>

<style>
:root{--g1:#0f2027;--g2:#203a43;--g3:#2c5364;--a:#4facfe;--b:#00f2fe}

body{
  margin:0;
  font-family:sans-serif;
  background:linear-gradient(135deg,var(--g1),var(--g2),var(--g3));
  color:#fff;
}

/* LOCK */
#lockScreen{
  position:fixed; inset:0;
  display:flex; align-items:center; justify-content:center;
  background:#000c;
}
.lockBox{
  background:#111;
  padding:30px;
  border-radius:15px;
  text-align:center;
}

/* INPUT + BUTTON */
input{padding:10px;border-radius:8px;border:none;text-align:center}
button{
  padding:10px;
  border:none;
  border-radius:8px;
  background:linear-gradient(45deg,var(--a),var(--b));
  color:#fff;
  cursor:pointer;
}

/* MAIN */
.container{padding:20px;max-width:1200px;margin:auto}

.tabs{
  display:flex;
  gap:10px;
  justify-content:center;
  margin-bottom:10px;
}

/* GRID FIXED */
.gallery{
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(150px,1fr));
  gap:12px;
}

.card{
  background:#111;
  padding:8px;
  border-radius:10px;
  position:relative;
  overflow:hidden;
}
.card:hover{transform:scale(1.05)}

/* 🔥 FIXED SIZE */
img,video{
  width:100%;
  height:200px;
  object-fit:cover;
  border-radius:8px;
  cursor:pointer;
}

/* SELECT */
.checkbox{
  position:absolute;
  top:5px;
  left:5px;
}

/* DOWNLOAD */
.download{
  display:block;
  margin-top:6px;
  text-align:center;
  background:#28a745;
  padding:6px;
  border-radius:6px;
  font-size:12px;
}

/* TOPBAR */
.topbar{text-align:center;margin-bottom:10px}

.hidden{display:none}

/* PREVIEW */
#preview{
  position:fixed;
  inset:0;
  background:#000e;
  display:none;
  align-items:center;
  justify-content:center;
  flex-direction:column;
  cursor:pointer;
}
#preview img,#preview video{
  max-width:90%;
  max-height:80%;
}

.counter{margin-top:5px}

</style>
</head>

<body>

<!-- LOCK -->
<div id="lockScreen">
  <div class="lockBox">
    <h2>🔒 Enter PIN</h2>
    <input type="password" id="pin">
    <br><br>
    <button onclick="unlock()">Unlock</button>
    <p id="error" style="color:red"></p>
  </div>
</div>

<!-- MAIN -->
<div class="container hidden" id="mainContent">

  <h1>⚡ Ultimate Gallery</h1>

  <div class="tabs">
    <button onclick="showSection('photos')">📸 Photos</button>
    <button onclick="showSection('videos')">🎥 Videos</button>
  </div>

  <div class="topbar">
    <button onclick="selectAll()">✔ Select All</button>
    <button onclick="clearAll()">❌ Clear</button>
    <button onclick="downloadZip()">⬇ Download ZIP</button>
    <div class="counter" id="counter">0 selected</div>
  </div>

  <div id="photos" class="gallery"></div>
  <div id="videos" class="gallery hidden"></div>

</div>

<!-- PREVIEW -->
<div id="preview" onclick="closePreview()">
  <div id="previewBox"></div>
</div>

<script>

/* PIN */
const correctPin="701309";

function unlock(){
  if(pin.value===correctPin){
    lockScreen.style.display='none';
    mainContent.classList.remove('hidden');
  } else {
    error.innerText='Wrong PIN';
  }
}

/* SWITCH */
function showSection(id){
  photos.classList.add('hidden');
  videos.classList.add('hidden');
  document.getElementById(id).classList.remove('hidden');
}

/* AUTO FILES */
const photoFiles=[];
const videoFiles=[];

for(let i=1;i<=20;i++){
  photoFiles.push(`image${i}.jpeg`);
}

for(let i=1;i<=7;i++){
  videoFiles.push(`video${i}.mp4`);
}

/* PREVIEW */
function preview(type,src){
  if(type==='img'){
    previewBox.innerHTML=`<img src="${src}">`;
  } else {
    previewBox.innerHTML=`<video controls autoplay><source src="${src}"></video>`;
  }
  preview.style.display='flex';
}

function closePreview(){
  preview.style.display='none';
}

/* FORCE DOWNLOAD */
function forceDownload(file){
  const a=document.createElement('a');
  a.href=file;
  a.download=file;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
}

/* CARD */
function createCard(file,type){
  const div=document.createElement('div');
  div.className='card';

  const media = type==='img'
    ? `<img src="${file}" onclick="preview('img','${file}')">`
    : `<video onclick="preview('video','${file}')" muted><source src="${file}"></video>`;

  div.innerHTML=`
    <input type="checkbox" class="checkbox" value="${file}" onchange="updateCount()">
    ${media}
    <button class="download" onclick="forceDownload('${file}')">Download</button>
  `;

  return div;
}

/* LOAD */
function load(){
  photoFiles.forEach(f=>photos.appendChild(createCard(f,'img')));
  videoFiles.forEach(f=>videos.appendChild(createCard(f,'video')));
}

/* SELECT */
function getSelected(){
  return [...document.querySelectorAll('.checkbox:checked')];
}

function updateCount(){
  counter.innerText=getSelected().length+' selected';
}

function selectAll(){
  document.querySelectorAll('.checkbox').forEach(cb=>cb.checked=true);
  updateCount();
}

function clearAll(){
  document.querySelectorAll('.checkbox').forEach(cb=>cb.checked=false);
  updateCount();
}

/* ZIP */
async function downloadZip(){
  const files=getSelected();
  if(files.length===0){alert('Select files');return;}

  const zip=new JSZip();

  for(const f of files){
    const res=await fetch(f.value);
    const blob=await res.blob();
    zip.file(f.value,blob);
  }

  zip.generateAsync({type:'blob'}).then(content=>{
    saveAs(content,'gallery.zip');
  });
}

load();

</script>

</body>
</html>
