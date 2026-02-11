// ==========================================
// Cloudflare Worker - 终极黑客终端聊天室（V12 极速响应版）
// 新特性：
// 1. LocalStorage 本地持久化：刷新瞬间显示历史记录，掩盖 KV 延迟
// 2. Markdown 简易渲染：支持 **粗体** 和 ~~删除线~~
// 3. 输入暂停轮询：打字时减少请求，优化性能
// ==========================================

const HTML_CONTENT = `<!DOCTYPE html><html lang="zh-CN"><head><meta charset="UTF-8"><meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no,viewport-fit=cover"><title>Global Link</title><style>*{box-sizing:border-box;margin:0;padding:0;outline:none;-webkit-tap-highlight-color:transparent}
:root{--bg-body:#000;--bg-card:#1c1c1e;--text-main:#fff;--text-sub:#8e8e93;--accent-color:#007aff;--msg-bg-me:#0a84ff;--msg-text-me:#fff;--msg-bg-other:#2c2c2e;--msg-text-other:#fff;--danger-color:#ff453a;--success-color:#30d158;--radius-box:24px;--radius-btn:12px}
body.light-mode{--bg-body:#f2f2f7;--bg-card:#fff;--text-main:#000;--text-sub:#8e8e93;--accent-color:#007aff;--msg-bg-me:#007aff;--msg-text-me:#fff;--msg-bg-other:#e5e5ea;--msg-text-other:#000}
body{background-color:var(--bg-body);color:var(--text-main);font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;height:100dvh;overflow:hidden;display:flex;justify-content:center;align-items:center;transition:background-color .3s,color .3s}
.main-container{width:100%;height:100%;background-color:var(--bg-card);display:flex;flex-direction:column;position:relative;max-width:600px;max-height:900px}
@media(min-width:601px){.main-container{height:95vh;border-radius:var(--radius-box);box-shadow:0 20px 60px rgba(0,0,0,.5);margin:2.5vh auto;border:1px solid rgba(255,255,255,.1)}}
#login-ui{flex:1;display:flex;flex-direction:column;justify-content:center;padding:30px}
h1{text-align:center;margin-bottom:40px;font-size:1.5rem;font-weight:700;letter-spacing:-.5px}
.input-group{margin-bottom:20px;position:relative}
.input-label{font-size:13px;color:var(--text-sub);margin-bottom:8px;display:block;font-weight:600}
input[type="text"],input[type="password"]{width:100%;background:var(--bg-body);border:1px solid transparent;color:var(--text-main);font-size:17px;padding:16px;border-radius:var(--radius-btn);transition:all .2s}
input:focus{border-color:var(--accent-color);box-shadow:0 0 0 3px rgba(0,122,255,.2)}
button{width:100%;padding:16px;font-size:17px;font-weight:600;border-radius:var(--radius-btn);border:none;background-color:var(--accent-color);color:#fff;cursor:pointer;transition:transform .1s,opacity .2s}
button:active{transform:scale(.98);opacity:.9}
button:disabled{background-color:var(--text-sub);cursor:not-allowed}
#chat-ui{display:none;flex-direction:column;height:100%;padding-bottom:env(safe-area-inset-bottom)}
.toolbar{padding:15px;display:flex;gap:8px;align-items:center;justify-content:space-between;background:var(--bg-card);border-bottom:1px solid rgba(0,0,0,.05);z-index:10;flex-wrap:wrap}
.btn-icon{width:38px;height:38px;border-radius:50%;background:var(--bg-body);color:var(--text-main);display:flex;align-items:center;justify-content:center;font-size:20px;flex-shrink:0;border:none}
.btn-text{width:auto;padding:8px 12px;border-radius:20px;font-size:13px;background:var(--bg-body);color:var(--accent-color)}
.btn-text:hover{background:rgba(0,0,0,.05)}
.btn-danger{color:var(--danger-color);background:rgba(255,69,58,.1)}
.btn-hard{color:var(--accent-color);font-weight:bold;border:1px solid var(--accent-color)}
#chat-log{flex:1;overflow-y:auto;padding:15px;background:var(--bg-card);scroll-behavior:smooth}
.msg-wrapper{margin-bottom:12px;display:flex;flex-direction:column}
.msg-wrapper.mine{align-items:flex-end}
.msg-content{max-width:80%;padding:10px 16px;border-radius:20px;font-size:16px;line-height:1.4;position:relative;word-wrap:break-word;white-space:pre-wrap}
.msg-wrapper.mine .msg-content{background-color:var(--msg-bg-me);color:var(--msg-text-me);border-bottom-right-radius:6px}
.msg-wrapper.other .msg-content{background-color:var(--msg-bg-other);color:var(--msg-text-other);border-bottom-left-radius:6px}
/* Markdown Styles */
.msg-content b{font-weight:800}
.msg-content em[style*="text-decoration"]{text-decoration:line-through;opacity:0.8}
.msg-meta{font-size:12px;color:var(--text-sub);margin-top:4px;display:flex;align-items:center;gap:6px}
.msg-wrapper.mine .msg-meta{flex-direction:row-reverse}
.input-area{padding:10px 15px env(safe-area-inset-bottom) 15px;background:var(--bg-card);border-top:1px solid rgba(0,0,0,.05);display:flex;align-items:center;gap:10px}
.input-wrapper{flex:1;position:relative;background:var(--bg-body);border-radius:25px;display:flex;align-items:center}
#chat-input{flex:1;background:transparent;border:none;padding:12px 15px;font-size:16px;color:var(--text-main);border-radius:25px}
.status-text{font-size:11px;color:var(--text-sub);padding:0 10px;white-space:nowrap}
.toast{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);background:rgba(0,0,0,.8);color:#fff;padding:10px 20px;border-radius:20px;font-size:14px;opacity:0;pointer-events:none;transition:opacity .3s;z-index:100}
.toast.show{opacity:1}
.sys-msg{text-align:center;font-size:12px;color:var(--text-sub);margin:20px 0}
@keyframes popIn{from{transform:scale(.9);opacity:0}to{transform:scale(1);opacity:1}}
.msg-content{animation:popIn .2s cubic-bezier(.175,.885,.32,1.275)}</style></head><body><div class="main-container"><div id="login-ui"><h1>Access Link</h1><form id="login-form" onsubmit="handleLogin(event)"><div class="input-group"><span class="input-label">Identity / 用户名</span><input type="text" id="username" autocomplete="off" placeholder="Agent Name"></div><div class="input-group"><span class="input-label">Passcode / 密钥</span><input type="password" id="password" autocomplete="off" placeholder="••••••••"></div><button type="submit" id="login-btn">Connect</button></form><div id="message-box" class="sys-msg" style="display:none;"></div></div><div id="chat-ui"><div class="toolbar"><button onclick="toggleTheme()" class="btn-icon" title="切换主题">☀</button><button onclick="refreshChat()" class="btn-text">⟳ 更新</button><button onclick="hardRefresh()" class="btn-text btn-hard">⟳ 强制重置</button><button onclick="clearChat()" class="btn-text btn-danger">☢ 清空</button></div><div id="chat-log"></div><div class="input-area"><div class="input-wrapper"><span style="padding-left:10px;color:var(--text-sub)">✎</span><input type="text" id="chat-input" placeholder="Type message..." autocomplete="off" oninput="handleInput()"></div><div id="save-status" class="status-text">Ready</div><button onclick="sendMessage()" class="btn-icon" style="background:var(--accent-color);width:44px;height:44px;border-radius:50%"><span style="font-size:20px;color:#fff">➤</span></button></div><div id="toast" class="toast">Action Successful</div></div></div><script>const PASSWORD="终端";const ADMIN_PASS="8888";const ACCESS_KEY="hacker_2024_key";const LOCAL_KEY="chat_history_v12";const loginUI=document.getElementById("login-ui");const chatUI=document.getElementById("chat-ui");const loginBtn=document.getElementById("login-btn");const msgBox=document.getElementById("message-box");const chatInput=document.getElementById("chat-input");const chatLog=document.getElementById("chat-log");const refreshBtn=document.getElementById("refresh-btn");const saveStatus=document.getElementById("save-status");let lastMsgCount=-1;let isLightMode=false;let timer=null;let typing=false;function toggleTheme(){isLightMode=!isLightMode;if(isLightMode)document.body.classList.add("light-mode");else document.body.classList.remove("light-mode")}function handleLogin(e){e.preventDefault();const user=document.getElementById("username").value.trim();const pass=document.getElementById("password").value.trim();if(user===PASSWORD&&pass===PASSWORD){loginBtn.innerText="Connecting...";msgBox.style.display="block";msgBox.innerText="Verifying Encryption Keys...";setTimeout(()=>{loginUI.style.display="none";chatUI.style.display="flex";chatInput.focus();loadMessages(true);startAutoRefresh()},1000)}else{const inputPw=document.getElementById("password");inputPw.style.borderColor="var(--danger-color)";setTimeout(()=>inputPw.style.borderColor="transparent",500)}}function startAutoRefresh(){if(timer)clearInterval(timer);timer=setInterval(()=>{if(!typing)loadMessages()},1000)}function handleInput(){typing=true;clearTimeout(timer.timerId);timer.timerId=setTimeout(()=>{typing=false},2000)}async function sendMessage(){const text=chatInput.value.trim();if(!text)return;const user=document.getElementById("username").value.trim()||"Agent";chatInput.disabled=true;chatInput.value="";chatInput.placeholder="Processing...";saveStatus.innerText="Saving to KV...";saveStatus.style.color="var(--accent-color);typing=true;let success=false;for(let i=0;i<3;i++){try{const res=await fetch("/api/chat",{method:"POST",headers:{"Content-Type":"application/json","X-Access-Key":ACCESS_KEY},body:JSON.stringify({user,text})});if(res.ok){const serverCheck=await res.json();if(serverCheck.status==="verified"){success=true;break}else{console.log("Verification failed, retrying...")}}}catch(err){if(i<2)await new Promise(r=>setTimeout(r,500))}}chatInput.disabled=false;chatInput.placeholder="Type message...";chatInput.focus();if(success){saveStatus.innerText="Saved ✓";saveStatus.style.color="var(--success-color)";setTimeout(()=>{saveStatus.innerText="Ready";saveStatus.style.color="var(--text-sub)"},2000);loadMessages(true)}else{chatInput.value=text;saveStatus.innerText="Failed";saveStatus.style.color="var(--danger-color)";alert("发送失败：无法确认 KV 存储成功。")}}chatInput.addEventListener("keypress",e=>{if(e.key==="Enter")sendMessage()});async function refreshChat(){const originalText=refreshBtn.innerText;refreshBtn.innerText="Updating...";refreshBtn.disabled=true;try{const res=await fetch("/api/chat?t="+Date.now(),{headers:{"X-Access-Key":ACCESS_KEY}});if(!res.ok)throw new Error("Network Error");const data=await res.json();renderMessages(data);saveLocal(data);showToast("更新成功")}catch(err){showToast("更新失败")}refreshBtn.innerText=originalText;refreshBtn.disabled=false}async function hardRefresh(){chatLog.innerHTML='<div class="sys-msg">正在强制重置...</div>';lastMsgCount=-1;localStorage.removeItem(LOCAL_KEY);await loadMessages(true);showToast("已强制重置")}async function clearChat(){const pwd=prompt("请输入管理员密码 (8888)：");if(pwd===ADMIN_PASS){if(confirm("确定要销毁所有聊天记录吗？")){try{const res=await fetch("/api/clear",{method:"POST",headers:{"Content-Type":"application/json","X-Access-Key":ACCESS_KEY},body:JSON.stringify({password:ADMIN_PASS})});if(res.ok){chatLog.innerHTML="";localStorage.removeItem(LOCAL_KEY);showToast("数据已销毁")}}catch(err){alert("网络错误")}}}}async function loadMessages(force){if(force){localStorage.removeItem(LOCAL_KEY);const res=await fetch("/api/chat?t="+Date.now(),{headers:{"X-Access-Key":ACCESS_KEY}});const data=await res.json();renderMessages(data);saveLocal(data);return}try{const localData=localStorage.getItem(LOCAL_KEY);if(localData){const parsed=JSON.parse(localData);if(parsed.length>0)renderMessages(parsed);lastMsgCount=parsed.length}const res=await fetch("/api/chat?t="+Date.now(),{headers:{"X-Access-Key":ACCESS_KEY}});const data=await res.json();if(data.length>lastMsgCount){renderMessages(data);saveLocal(data)}else if(localData&&data.length<lastMsgCount){renderMessages(data);saveLocal(data)}}catch(err){}}function saveLocal(data){try{localStorage.setItem(LOCAL_KEY,JSON.stringify(data))}catch(e){console.log("Storage full")}}function parseMarkdown(text){let html=text.replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;");html=html.replace(/\\*\\*(.*?)\\*\\*/g,"<b>$1</b>");html=html.replace(/~~(.*?)~~/g,"<em style='text-decoration:line-through'>$1</em>");return html}function renderMessages(data){chatLog.innerHTML="";if(data.length===0){chatLog.innerHTML='<div class="sys-msg">-- 暂无记录 --</div>';return}data.forEach(function(msg){const currentUser=document.getElementById("username").value.trim()||"Unknown";const isMyMsg=msg.user===currentUser;const wrapper=document.createElement("div");wrapper.className="msg-wrapper "+(isMyMsg?"mine":"other");const content=document.createElement("div");content.className="msg-content";content.innerHTML=parseMarkdown(msg.text);const meta=document.createElement("div");meta.className="msg-meta";meta.innerHTML=\`\${msg.user} • \${msg.time} <span class="user-ip">@\${msg.ip}</span>\`;wrapper.appendChild(content);wrapper.appendChild(meta);chatLog.appendChild(wrapper)});chatLog.scrollTop=chatLog.scrollHeight}function showToast(msg){const t=document.getElementById("toast");t.innerText=msg;t.classList.add("show");setTimeout(()=>t.classList.remove("show"),2000)}</script></body></html>`;

export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const ACCESS_KEY = "hacker_2024_key";

    if (url.pathname === "/") {
      return new Response(HTML_CONTENT, {
        headers: {
          "content-type": "text/html;charset=UTF-8",
          "Cache-Control": "no-cache, no-store, max-age=0"
        }
      });
    }

    if (url.pathname === "/api/chat" && request.method === "GET") {
      const clientKey = request.headers.get("X-Access-Key");
      if (clientKey !== ACCESS_KEY) return new Response("Forbidden", { status: 403 });
      try {
        const data = await env.CHAT_DB.get("messages", { type: "json" });
        const messages = data || [];
        return new Response(JSON.stringify(messages), { 
          headers: { 
            "content-type": "application/json",
            "Cache-Control": "no-cache, no-store, max-age=0, must-revalidate", 
            "Pragma": "no-cache",
            "Expires": "0"
          } 
        });
      } catch (err) {
        return new Response("Error", { status: 500 });
      }
    }

    if (url.pathname === "/api/chat" && request.method === "POST") {
      const clientKey = request.headers.get("X-Access-Key");
      if (clientKey !== ACCESS_KEY) return new Response("Forbidden", { status: 403 });

      try {
        const body = await request.json();
        var user = body.user;
        var text = body.text;
        var ip = request.headers.get("CF-Connecting-IP") || "Unknown";
        var time = new Date().toLocaleTimeString("zh-CN", { hour12: false });

        var messages = (await env.CHAT_DB.get("messages", { type: "json" })) || [];
        messages.push({ user, text, ip, time });
        if (messages.length > 50) messages = messages.slice(messages.length - 50);

        await env.CHAT_DB.put("messages", JSON.stringify(messages));

        let verified = false;
        for(let v=0; v<3; v++) {
            const checkData = await env.CHAT_DB.get("messages", { type: "json" });
            if (checkData && checkData.length > 0) {
                const lastMsg = checkData[checkData.length - 1];
                if (lastMsg.text === text && lastMsg.user === user) {
                    verified = true;
                    break; 
                }
            }
            if (v < 2) await new Promise(r => setTimeout(r, 100));
        }

        if (!verified) {
            return new Response("KV Verification Failed", { status: 500 });
        }

        return new Response(JSON.stringify({ status: "verified" }), { status: 200 });
      } catch (err) {
        console.error(err);
        return new Response("Error", { status: 500 });
      }
    }

    if (url.pathname === "/api/clear" && request.method === "POST") {
      const clientKey = request.headers.get("X-Access-Key");
      if (clientKey !== ACCESS_KEY) return new Response("Forbidden", { status: 403 });
      try {
        const body = await request.json();
        if (body.password === "8888") {
          await env.CHAT_DB.put("messages", JSON.stringify([]));
          const verify = await env.CHAT_DB.get("messages", { type: "json" });
          if (verify.length === 0) return new Response("OK", { status: 200 });
          else return new Response("Clear Failed", { status: 500 });
        } else {
          return new Response("Forbidden", { status: 403 });
        }
      } catch (err) {
        return new Response("Error", { status: 500 });
      }
    }

    return new Response("404 Not Found", { status: 404 });
  }
};

