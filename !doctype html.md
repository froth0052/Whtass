<!doctype html>  
<html lang="en" class="h-full">  
<head>  
  <meta charset="UTF-8">  
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">  
  <title>Write Together | Mobile Edition</title>  
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>  
  <link href="https://fonts.googleapis.com/css2?family=Crimson+Pro:wght@400;500;600&family=DM+Sans:wght@400;500;600&display=swap" rel="stylesheet">  
  <style>  
    :root {  
      --bg-color: #1a1a2e;  
      --surface-color: #16213e;  
      --text-color: #edf2f7;  
      --primary-color: #e94560;  
      --secondary-color: #0f3460;  
    }  
    body {   
      font-family: 'DM Sans', sans-serif;   
      background: var(--bg-color);   
      color: var(--text-color);   
      -webkit-tap-highlight-color: transparent;  
    }  
    .writing-font { font-family: 'Crimson Pro', serif; }  
    .glow-border { box-shadow: 0 0 20px rgba(233, 69, 96, 0.15); }  
    /* Fix for iPad Audio Player styling */  
    audio { height: 44px; border-radius: 8px; }  
    .tab-btn { padding: 1rem; flex: 1; text-align: center; border-bottom: 3px solid transparent; opacity: 0.6; }  
    .tab-btn.active { opacity: 1; border-bottom-color: var(--primary-color); }  
    input, textarea { font-size: 16px !important; } /* Prevents iOS auto-zoom on focus */  
  </style>  
</head>  
<body class="h-full">  
  
  <script>  
    const STORAGE_KEY = 'write_together_ipad';  
    window.dataSdk = {  
      init: function(handler) {  
        this.handler = handler;  
        this.refresh();  
        window.addEventListener('storage', () => this.refresh());  
      },  
      refresh: function() {  
        const data = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');  
        if (this.handler && this.handler.onDataChanged) this.handler.onDataChanged(data);  
      },  
      create: async function(item) {  
        const data = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');  
        item.__backendId = Date.now().toString() + Math.random().toString(36).substr(2, 5);  
        data.push(item);  
        localStorage.setItem(STORAGE_KEY, JSON.stringify(data));  
        this.refresh();  
        return { isOk: true };  
      },  
      delete: async function(itemToDelete) {  
        let data = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');  
        data = data.filter(item => item.__backendId !== itemToDelete.__backendId);  
        localStorage.setItem(STORAGE_KEY, JSON.stringify(data));  
        this.refresh();  
        return { isOk: true };  
      }  
    };  
  </script>  
  
  <div id="app-wrapper" class="min-h-screen w-full pb-10">  
    <div class="max-w-2xl mx-auto px-4 py-6">  
        
      <div id="auth-section">  
        <div class="p-8 rounded-2xl glow-border bg-[#16213e] text-center">  
          <h1 class="text-3xl font-semibold writing-font mb-6">Write Together</h1>  
          <input type="text" id="login-user" placeholder="Your Name" class="w-full p-4 rounded-xl mb-4 bg-[#1a1a2e] border-none text-white">  
          <button id="enter-btn" class="w-full p-4 rounded-xl font-bold bg-[#e94560]">Start Writing</button>  
        </div>  
      </div>  
  
      <div id="main-section" class="hidden">  
        <nav class="flex mb-6 bg-[#16213e] rounded-xl overflow-hidden">  
          <button class="tab-btn active" data-tab="writing">✍️ Story</button>  
          <button class="tab-btn" data-tab="messages">🎤 Voice</button>  
        </nav>  
  
        <div id="writing-tab" class="tab-content">  
          <div class="p-4 rounded-2xl bg-[#16213e] mb-6">  
            <textarea id="new-entry" placeholder="What happens next?" rows="4" class="w-full p-3 rounded-xl bg-[#1a1a2e] text-lg writing-font mb-4 border-none"></textarea>  
            <div id="photo-preview-container" class="hidden mb-4">  
              <img id="photo-preview" class="w-full rounded-xl">  
            </div>  
            <div class="flex justify-between items-center">  
              <button onclick="document.getElementById('photo-input').click()" class="text-sm">📷 Image</button>  
              <input type="file" id="photo-input" accept="image/*" class="hidden">  
              <button id="add-entry-btn" class="px-8 py-3 rounded-xl font-bold bg-[#e94560]">Post</button>  
            </div>  
          </div>  
          <div id="entries-container" class="space-y-4"></div>  
        </div>  
  
        <div id="messages-tab" class="tab-content hidden">  
          <div class="p-8 rounded-2xl bg-[#16213e] text-center mb-6">  
            <div class="flex justify-center gap-4">  
              <button id="rec-start" class="w-20 h-20 rounded-full bg-red-500 flex items-center justify-center text-2xl">🎤</button>  
              <button id="rec-stop" disabled class="w-20 h-20 rounded-full bg-gray-600 flex items-center justify-center text-2xl opacity-50">⏹️</button>  
            </div>  
            <p id="rec-status" class="mt-4 text-sm opacity-50 font-medium">Tap Mic to Record</p>  
          </div>  
          <div id="messages-container" class="space-y-4"></div>  
        </div>  
      </div>  
    </div>  
  </div>  
  
  <script>  
    let currentUser = null;  
    let entries = [], messages = [];  
    let mediaRecorder = null;  
    let audioChunks = [];  
    let selectedImage = null;  
  
    document.getElementById('enter-btn').addEventListener('click', () => {  
      const name = document.getElementById('login-user').value.trim();  
      if (!name) return;  
      currentUser = name;  
      document.getElementById('auth-section').classList.add('hidden');  
      document.getElementById('main-section').classList.remove('hidden');  
    });  
  
    document.querySelectorAll('.tab-btn').forEach(btn => {  
      btn.addEventListener('click', () => {  
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));  
        document.querySelectorAll('.tab-content').forEach(c => c.classList.add('hidden'));  
        btn.classList.add('active');  
        document.getElementById(`${btn.dataset.tab}-tab`).classList.remove('hidden');  
      });  
    });  
  
    document.getElementById('photo-input').addEventListener('change', e => {  
      const reader = new FileReader();  
      reader.onload = ev => {  
        selectedImage = ev.target.result;  
        document.getElementById('photo-preview').src = selectedImage;  
        document.getElementById('photo-preview-container').classList.remove('hidden');  
      };  
      reader.readAsDataURL(e.target.files[0]);  
    });  
  
    document.getElementById('add-entry-btn').addEventListener('click', async () => {  
      const content = document.getElementById('new-entry').value;  
      if(!content && !selectedImage) return;  
      await window.dataSdk.create({  
        type: 'entry', author: currentUser, content, image: selectedImage, timestamp: new Date().toISOString()  
      });  
      document.getElementById('new-entry').value = '';  
      selectedImage = null;  
      document.getElementById('photo-preview-container').classList.add('hidden');  
    });  
  
    const recStart = document.getElementById('rec-start');  
    const recStop = document.getElementById('rec-stop');  
    const recStatus = document.getElementById('rec-status');  
  
    recStart.addEventListener('click', async () => {  
      try {  
        const stream = await navigator.mediaDevices.getUserMedia({ audio: true });  
        const mimeType = MediaRecorder.isTypeSupported('audio/mp4') ? 'audio/mp4' : 'audio/webm';  
        mediaRecorder = new MediaRecorder(stream, { mimeType });  
        audioChunks = [];  
        mediaRecorder.ondataavailable = e => audioChunks.push(e.data);  
        mediaRecorder.onstop = async () => {  
          const blob = new Blob(audioChunks, { type: mimeType });  
          const reader = new FileReader();  
          reader.readAsDataURL(blob);  
          reader.onloadend = async () => {  
            await window.dataSdk.create({  
              type: 'message', author: currentUser, audio: reader.result, timestamp: new Date().toISOString()  
            });  
          };  
        };  
        mediaRecorder.start();  
        recStart.disabled = true; recStop.disabled = false;  
        recStop.classList.remove('opacity-50');  
        recStatus.innerText = "Recording...";  
      } catch (err) { alert("Enable Microphone in iPad Settings"); }  
    });  
  
    recStop.addEventListener('click', () => {  
      mediaRecorder.stop();  
      recStart.disabled = false; recStop.disabled = true;  
      recStop.classList.add('opacity-50');  
      recStatus.innerText = "Saved!";  
    });  
  
    window.dataSdk.init({  
      onDataChanged: (data) => {  
        entries = data.filter(d => d.type === 'entry').sort((a,b) => new Date(b.timestamp) - new Date(a.timestamp));  
        messages = data.filter(d => d.type === 'message').sort((a,b) => new Date(b.timestamp) - new Date(a.timestamp));  
        render();  
      }  
    });  
  
    function render() {  
      document.getElementById('entries-container').innerHTML = entries.map(e => `  
        <div class="p-5 rounded-2xl bg-[#16213e]">  
          <div class="flex justify-between text-[10px] opacity-50 mb-3"><b>${e.author}</b><span>${new Date(e.timestamp).toLocaleTimeString()}</span></div>  
          ${e.image ? `<img src="${e.image}" class="w-full rounded-xl mb-3">` : ''}  
          <p class="writing-font text-xl leading-relaxed">${e.content}</p>  
          <button onclick="del('${e.__backendId}')" class="mt-4 opacity-20 text-[10px]">Delete</button>  
        </div>  
      `).join('');  
  
      document.getElementById('messages-container').innerHTML = messages.map(m => `  
        <div class="p-4 rounded-2xl bg-[#16213e]">  
          <p class="text-[10px] mb-2 font-bold color-[#e94560]">${m.author}</p>  
          <audio controls src="${m.audio}" class="w-full"></audio>  
        </div>  
      `).join('');  
    }  
    window.del = id => window.dataSdk.delete({__backendId: id});  
  </script>  
</body>  
</html>  
