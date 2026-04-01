<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cognitive AI | Kastav</title>

<style>
body {
  margin:0;
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(135deg,#1f1c2c,#928dab);
  color:white;
}
.container { max-width:500px; margin:auto; padding:15px;}
.header { text-align:center; font-size:22px; font-weight:bold;}

.card {
  background: rgba(255,255,255,0.1);
  backdrop-filter: blur(10px);
  border-radius:15px;
  padding:10px;
  margin:10px 0;
}

.chat-box { height:300px; overflow-y:auto;}

.msg {
  padding:10px;
  margin:5px 0;
  border-radius:10px;
  max-width:80%;
  animation: fadeIn 0.3s ease-in;
}
.user { background:#4facfe; margin-left:auto;}
.ai { background:#43e97b; color:black;}

input, button {
  width:100%;
  padding:10px;
  margin-top:5px;
  border:none;
  border-radius:10px;
}

button { background:white; font-weight:bold;}

.topbar { display:flex; gap:5px;}
.small-btn { flex:1;}

button:hover { transform: scale(1.05); transition:0.2s;}

@keyframes fadeIn {
  from {opacity:0; transform:translateY(10px);}
  to {opacity:1; transform:translateY(0);}
}
</style>
</head>

<body>
<div class="container">

<div class="header">🧠 Cognitive AI</div>

<div class="card topbar">
<button class="small-btn" onclick="shareApp()">🔗</button>
<button class="small-btn" onclick="downloadData()">⬇️</button>
</div>

<div class="card">
<input id="goal" placeholder="🎯 Your Goal">
<button onclick="saveGoal()">Save Goal</button>
</div>

<div class="card chat-box" id="chat"></div>

<div class="card">
<input id="input" placeholder="Type message...">
<button onclick="send()">Send</button>
<button onclick="voice()">🎤 Speak</button>
</div>

</div>

<!-- FIREBASE -->
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js"></script>

<script>

// 🔥 STEP 1: ADD YOUR FIREBASE CONFIG HERE
const firebaseConfig = {
  apiKey: "PASTE_FIREBASE_KEY",
  authDomain: "YOUR_DOMAIN",
  projectId: "YOUR_PROJECT_ID"
};

firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();

// AUTO LOGIN (ANONYMOUS)
auth.signInAnonymously();

// UI
function addMessage(text, type){
 let div=document.createElement("div");
 div.className="msg "+type;
 div.innerText=text;
 chat.appendChild(div);
 chat.scrollTop=chat.scrollHeight;
}

// SAVE GOAL
function saveGoal(){
 localStorage.setItem("goal", goal.value);
 alert("Goal saved!");
}

// SHARE
function shareApp(){
 navigator.clipboard.writeText(window.location.href);
 alert("Link copied!");
}

// DOWNLOAD
function downloadData(){
 let data=localStorage.getItem("memory");
 let blob=new Blob([data],{type:"application/json"});
 let a=document.createElement("a");
 a.href=URL.createObjectURL(blob);
 a.download="data.json";
 a.click();
}

// VOICE
function voice(){
 let rec=new (window.SpeechRecognition||window.webkitSpeechRecognition)();
 rec.onresult=e=> input.value=e.results[0][0].transcript;
 rec.start();
}

// CLOUD SAVE
async function saveMemory(user, ai){
 let uid=auth.currentUser.uid;
 await db.collection("users").doc(uid).collection("memory").add({
  user, ai, time:Date.now()
 });
}

// LOAD MEMORY
async function loadMemory(){
 let uid=auth.currentUser.uid;
 let snap=await db.collection("users").doc(uid)
 .collection("memory").orderBy("time").get();

 let memory=[];
 snap.forEach(d=>memory.push(d.data()));
 return memory;
}

// 🔥 AI FUNCTION (OPENROUTER)
async function cognitiveAI(input){

 let memory=await loadMemory();
 let goal=localStorage.getItem("goal")||"";

 let context=memory.slice(-5).map(m=>m.user+" "+m.ai).join("\n");

 let response=await fetch("https://openrouter.ai/api/v1/chat/completions",{
  method:"POST",
  headers:{
   "Authorization":"Bearer YOUR_API_KEY",
   "Content-Type":"application/json"
  },
  body:JSON.stringify({
   model:"mistralai/mistral-7b-instruct",
   messages:[
    {role:"system",content:`You are a human-like cognitive AI. Goal:${goal} Context:${context}`},
    {role:"user",content:input}
   ]
  })
 });

 let data=await response.json();
 let reply=data.choices[0].message.content;

 await saveMemory(input, reply);

 return "🧠 "+reply+"\n\n— Cognitive AI";
}

// SEND
async function send(){
 let text=input.value;
 if(!text) return;

 addMessage(text,"user");
 addMessage("Thinking...","ai");

 let reply=await cognitiveAI(text);

 chat.lastChild.remove();
 addMessage(reply,"ai");

 input.value="";
 input.focus();
}

// ENTER KEY
input.addEventListener("keypress",e=>{
 if(e.key==="Enter") send();
});

</script>
</body>
</html>
