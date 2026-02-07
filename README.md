<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Universal AI Chat v2</title>

<!-- Tailwind -->
<script src="https://cdn.tailwindcss.com"></script>
<!-- Markdown Renderer -->
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>

<style>
body.dark { background:#0f172a; color:white; }
body.light { background:#f1f5f9; color:black; }

.chat-bubble {
  animation: fadeIn 0.2s ease-in-out;
}
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(5px); }
  to { opacity: 1; transform: translateY(0); }
}
.btn { transition: transform 0.1s ease-in-out; }
.btn:active { transform: scale(0.95); }

.markdown-body p { margin-bottom: 6px; }
</style>
</head>

<body class="dark">

<div class="flex h-screen">

<!-- ===== SIDEBAR ===== -->
<div class="w-64 bg-slate-900 border-r border-slate-700 p-4 hidden md:block">
  <div class="flex justify-between items-center mb-4">
    <h2 class="text-lg font-bold">Chats</h2>
    <button id="newChatBtn"
      class="btn border border-cyan-400 text-cyan-400 px-3 py-1 rounded-lg hover:bg-cyan-400 hover:text-black">
      + New
    </button>
  </div>

  <div id="chatList" class="space-y-2 overflow-y-auto h-[78vh]"></div>

  <div class="mt-4 space-y-2">
    <button id="openSettings"
      class="btn w-full border border-purple-400 text-purple-400 px-3 py-2 rounded-lg hover:bg-purple-400 hover:text-black">
      Settings
    </button>

    <button id="toggleTheme"
      class="btn w-full border border-yellow-400 text-yellow-400 px-3 py-2 rounded-lg">
      Toggle Theme
    </button>

    <button id="exportChat"
      class="btn w-full border border-green-400 text-green-400 px-3 py-2 rounded-lg">
      Export Chat
    </button>

    <button id="clearChat"
      class="btn w-full border border-red-400 text-red-400 px-3 py-2 rounded-lg">
      Clear Chat
    </button>
  </div>
</div>

<!-- ===== MOBILE SIDEBAR ===== -->
<button id="mobileMenuBtn"
class="md:hidden fixed top-4 left-4 btn border border-cyan-400 text-cyan-400 px-3 py-1 rounded-lg">
☰
</button>

<div id="mobileSidebar" class="fixed inset-0 bg-slate-900 z-50 hidden">
  <div class="p-4">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-lg font-bold">Chats</h2>
      <button id="closeMobile" class="text-red-400">✕</button>
    </div>

    <div id="mobileChatList" class="space-y-2 overflow-y-auto h-[75vh]"></div>

    <div class="mt-4 space-y-2">
      <button id="openSettingsMobile"
        class="btn w-full border border-purple-400 text-purple-400 px-3 py-2 rounded-lg">
        Settings
      </button>
      <button id="toggleThemeMobile"
        class="btn w-full border border-yellow-400 text-yellow-400 px-3 py-2 rounded-lg">
        Toggle Theme
      </button>
    </div>
  </div>
</div>

<!-- ===== MAIN CHAT ===== -->
<div class="flex-1 flex flex-col">

<header class="bg-slate-800 p-4 border-b border-slate-700 flex justify-between items-center">
  <h1 class="text-xl font-bold">Universal AI Chat v2</h1>
</header>

<div id="chatContainer" class="flex-1 overflow-y-auto p-4 space-y-4"></div>

<div class="p-4 bg-slate-900 border-t border-slate-700">
  <div class="flex gap-2 items-center">
    <textarea id="userInput"
      class="flex-1 bg-slate-800 border border-slate-700 rounded-lg p-2 resize-none focus:outline-none"
      rows="1" placeholder="Type your message..."></textarea>

    <button id="voiceBtn"
      class="btn border border-blue-400 text-blue-400 px-3 py-2 rounded-lg">
      🎤
    </button>

    <button id="sendBtn"
      class="btn border border-green-400 text-green-400 px-4 py-2 rounded-lg hover:bg-green-400 hover:text-black">
      Send
    </button>
  </div>
</div>

</div>
</div>

<!-- ===== SETTINGS MODAL ===== -->
<div id="settingsModal" class="fixed inset-0 bg-black bg-opacity-60 hidden flex items-center justify-center">
<div class="bg-slate-900 p-6 rounded-xl w-96 border border-slate-700">
  <h2 class="text-lg font-bold mb-4">Settings</h2>

  <label class="block mb-2">OpenRouter API Key</label>
  <input id="apiKeyInput" type="password"
    class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 mb-3" />

  <label class="block mb-2">Model Name</label>
  <input id="modelInput" type="text"
    value="google/gemini-2.0-flash-lite-preview-02-05"
    class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 mb-3" />

  <label class="block mb-2">System Prompt</label>
  <textarea id="systemPrompt"
    class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 mb-4"
    rows="3">You are a helpful AI assistant.</textarea>

  <div class="flex justify-end gap-2">
    <button id="closeSettings"
      class="btn border border-red-400 text-red-400 px-3 py-1 rounded-lg">
      Cancel
    </button>

    <button id="saveSettings"
      class="btn border border-cyan-400 text-cyan-400 px-3 py-1 rounded-lg">
      Save
    </button>
  </div>
</div>
</div>

<!-- ================= SCRIPT (END OF BODY) ================= -->
<script>
const chatContainer = document.getElementById("chatContainer");
const chatList = document.getElementById("chatList");
const mobileChatList = document.getElementById("mobileChatList");
const userInput = document.getElementById("userInput");
const sendBtn = document.getElementById("sendBtn");
const voiceBtn = document.getElementById("voiceBtn");
const newChatBtn = document.getElementById("newChatBtn");
const clearChat = document.getElementById("clearChat");
const exportChat = document.getElementById("exportChat");
const toggleTheme = document.getElementById("toggleTheme");
const toggleThemeMobile = document.getElementById("toggleThemeMobile");

const settingsModal = document.getElementById("settingsModal");
const openSettings = document.getElementById("openSettings");
const openSettingsMobile = document.getElementById("openSettingsMobile");
const closeSettings = document.getElementById("closeSettings");
const saveSettings = document.getElementById("saveSettings");
const apiKeyInput = document.getElementById("apiKeyInput");
const modelInput = document.getElementById("modelInput");
const systemPrompt = document.getElementById("systemPrompt");

const mobileMenuBtn = document.getElementById("mobileMenuBtn");
const mobileSidebar = document.getElementById("mobileSidebar");
const closeMobile = document.getElementById("closeMobile");

let currentChatId = null;

/* ===== THEME ===== */
function loadTheme(){
  const t = localStorage.getItem("theme") || "dark";
  document.body.className = t;
}
function switchTheme(){
  const now = document.body.classList.contains("dark") ? "light":"dark";
  localStorage.setItem("theme", now);
  loadTheme();
}
toggleTheme.onclick = switchTheme;
toggleThemeMobile.onclick = switchTheme;
loadTheme();

/* ===== STORAGE ===== */
function getChats(){
  return JSON.parse(localStorage.getItem("chats") || "[]");
}
function saveChats(c){ localStorage.setItem("chats", JSON.stringify(c)); }

/* ===== NEW CHAT ===== */
function createNewChat(){
  const chats = getChats();
  const id = Date.now().toString();
  chats.unshift({id, title:"New Chat", messages:[]});
  saveChats(chats);
  currentChatId = id;
  renderChatList();
  chatContainer.innerHTML = "";
}
newChatBtn.onclick = createNewChat;

/* ===== RENDER CHAT LIST ===== */
function renderChatList(){
  const chats = getChats();
  chatList.innerHTML = "";
  mobileChatList.innerHTML = "";

  chats.forEach(chat=>{
    const btn = document.createElement("button");
    btn.textContent = chat.title;
    btn.className = "w-full text-left px-3 py-2 rounded-lg border border-slate-700 hover:bg-slate-800 btn";
    btn.onclick = ()=>loadChat(chat.id);

    chatList.appendChild(btn);
    mobileChatList.appendChild(btn.cloneNode(true));
  });
}

/* ===== LOAD CHAT ===== */
function loadChat(id){
  const chats = getChats();
  const chat = chats.find(c=>c.id===id);
  if(!chat) return;

  currentChatId = id;
  chatContainer.innerHTML = "";

  chat.messages.forEach(m=>{
    addMessage(m.role, m.content, false);
  });

  mobileSidebar.classList.add("hidden");
}

/* ===== ADD MESSAGE (WITH MARKDOWN) ===== */
function addMessage(role, text, save=true){
  const div = document.createElement("div");
  div.className = "chat-bubble max-w-[80%] p-3 rounded-xl markdown-body " +
    (role==="user" ? "bg-cyan-600 ml-auto text-white" : "bg-slate-800 text-white");

  div.innerHTML = marked.parse(text);
  chatContainer.appendChild(div);
  chatContainer.scrollTop = chatContainer.scrollHeight;

  if(save && currentChatId){
    const chats = getChats();
    const chat = chats.find(c=>c.id===currentChatId);
    chat.messages.push({role, content:text});

    if(chat.messages.length===1 && role==="user"){
      chat.title = text.substring(0,25);
    }
    saveChats(chats);
    renderChatList();
  }
}

/* ===== VOICE INPUT ===== */
voiceBtn.onclick = ()=>{
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  if(!SpeechRecognition){ alert("Voice not supported"); return; }

  const rec = new SpeechRecognition();
  rec.lang = "en-US";
  rec.start();

  rec.onresult = e=>{
    userInput.value = e.results[0][0].transcript;
  };
};

/* ===== SEND MESSAGE ===== */
sendBtn.onclick = async ()=>{
  const text = userInput.value.trim();
  if(!text) return;

  if(!currentChatId) createNewChat();

  addMessage("user", text);
  userInput.value = "";

  const apiKey = localStorage.getItem("openrouter_api_key");
  const model = localStorage.getItem("ai_model");
  const sys = localStorage.getItem("system_prompt") || "You are a helpful AI.";

  if(!apiKey){
    addMessage("assistant","Please set your OpenRouter API Key in Settings.");
    return;
  }

  const placeholder = document.createElement("div");
  placeholder.className = "chat-bubble max-w-[80%] p-3 rounded-xl bg-slate-800";
  placeholder.textContent = "";
  chatContainer.appendChild(placeholder);

  try{
    const res = await fetch("https://openrouter.ai/api/v1/chat/completions",{
      method:"POST",
      headers:{
        "Authorization":`Bearer ${apiKey}`,
        "Content-Type":"application/json"
      },
      body:JSON.stringify({
        model:model,
        messages:[
          {role:"system", content:sys},
          {role:"user", content:text}
        ]
      })
    });

    const data = await res.json();
    const reply = data.choices[0].message.content;

    // Typewriter effect
    placeholder.innerHTML = "";
    let i=0;
    const interval = setInterval(()=>{
      placeholder.innerHTML = marked.parse(reply.substring(0,i));
      i++;
      if(i>reply.length){
        clearInterval(interval);
        addMessage("assistant", reply); // save final
        placeholder.remove();
      }
    },10);

  }catch(err){
    placeholder.remove();
    addMessage("assistant","Error connecting to AI. Check API Key.");
  }
};

/* ===== SETTINGS ===== */
apiKeyInput.value = localStorage.getItem("openrouter_api_key") || "";
modelInput.value = localStorage.getItem("ai_model") || "google/gemini-2.0-flash-lite-preview-02-05";
systemPrompt.value = localStorage.getItem("system_prompt") || "You are a helpful AI assistant.";

openSettings.onclick = ()=>settingsModal.classList.remove("hidden");
openSettingsMobile.onclick = ()=>settingsModal.classList.remove("hidden");
closeSettings.onclick = ()=>settingsModal.classList.add("hidden");

saveSettings.onclick = ()=>{
  localStorage.setItem("openrouter_api_key", apiKeyInput.value);
  localStorage.setItem("ai_model", modelInput.value);
  localStorage.setItem("system_prompt", systemPrompt.value);
  settingsModal.classList.add("hidden");
};

/* ===== EXPORT CHAT ===== */
exportChat.onclick = ()=>{
  if(!currentChatId) return alert("No chat selected");
  const chats = getChats();
  const chat = chats.find(c=>c.id===currentChatId);

  let text = chat.messages.map(m=>`${m.role}: ${m.content}`).join("\n\n");
  const blob = new Blob([text], {type:"text/plain"});
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "chat.txt";
  a.click();
};

/* ===== CLEAR CHAT ===== */
clearChat.onclick = ()=>{
  if(!currentChatId) return;
  const chats = getChats();
  const chat = chats.find(c=>c.id===currentChatId);
  chat.messages = [];
  saveChats(chats);
  chatContainer.innerHTML = "";
};

/* ===== MOBILE SIDEBAR ===== */
mobileMenuBtn.onclick = ()=>mobileSidebar.classList.remove("hidden");
closeMobile.onclick = ()=>mobileSidebar.classList.add("hidden");

/* INIT */
renderChatList();
</script>
</body>
</html>
