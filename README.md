# security
.
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hyper OS: Secure Persistent Edition</title>
    <style>
        /* --- CORE STYLES --- */
        :root {
            --bg-dark: #050505;
            --glass: rgba(20, 25, 35, 0.7);
            --glass-border: 1px solid rgba(255, 255, 255, 0.1);
            --blur: blur(20px);
            --accent: #00f3ff;
            --danger: #ff2a2a;
            --font: 'Segoe UI', system-ui, sans-serif;
            --mono: 'Consolas', monospace;
        }

        * { box-sizing: border-box; user-select: none; }
        body { margin: 0; overflow: hidden; background: #000; font-family: var(--font); color: white; height: 100vh; }

        /* --- WALLPAPER --- */
        #wallpaper {
            position: absolute; inset: 0; z-index: 0;
            background-size: cover; background-position: center;
            transition: 0.5s;
        }

        /* --- LOCK SCREEN --- */
        #lock-screen {
            position: fixed; inset: 0; z-index: 10000;
            background: rgba(0,0,0,0.9);
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            transition: opacity 0.5s;
        }
        .login-box { text-align: center; width: 300px; }
        .avatar { width: 100px; height: 100px; border-radius: 50%; background: #333; margin: 0 auto 20px; border: 2px solid var(--accent); background-image: url('https://cdn-icons-png.flaticon.com/512/3135/3135715.png'); background-size: cover; }
        .pass-input {
            width: 100%; padding: 10px; background: rgba(255,255,255,0.1); border: 1px solid #333;
            color: white; text-align: center; font-size: 18px; border-radius: 5px; outline: none;
            margin-top: 10px;
        }
        .pass-input:focus { border-color: var(--accent); }

        /* --- DESKTOP --- */
        #desktop { position: absolute; inset: 0; z-index: 10; padding: 20px; display: flex; flex-direction: column; flex-wrap: wrap; align-content: flex-start; gap: 10px; height: 90vh; }

        .icon {
            width: 90px; height: 100px; display: flex; flex-direction: column; align-items: center;
            cursor: pointer; border-radius: 5px; text-align: center; transition: 0.2s;
        }
        .icon:hover { background: rgba(255,255,255,0.1); }
        .icon img { width: 50px; height: 50px; margin-bottom: 5px; filter: drop-shadow(0 2px 5px black); }
        .icon span { font-size: 12px; text-shadow: 0 1px 2px black; word-break: break-word; }

        /* --- WINDOWS --- */
        .window {
            position: absolute; width: 500px; height: 400px;
            background: #1e1e1e; border: 1px solid #444; border-radius: 8px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.8);
            display: flex; flex-direction: column; overflow: hidden;
            top: 50px; left: 50px; opacity: 0; transform: scale(0.95);
            transition: opacity 0.2s, transform 0.2s;
        }
        .window.open { opacity: 1; transform: scale(1); }

        .titlebar {
            height: 35px; background: #2d2d2d; display: flex; align-items: center; justify-content: space-between; padding: 0 10px; cursor: grab;
        }
        .win-controls button { width: 12px; height: 12px; border-radius: 50%; border: none; margin-left: 5px; cursor: pointer; }
        .btn-close { background: #ff5555; } .btn-min { background: #ffbd44; } .btn-max { background: #00ca4e; }

        .win-content { flex: 1; background: #252526; color: #ccc; overflow: hidden; position: relative; }
        
        /* App Specifics */
        .editor-area { width: 100%; height: 100%; padding: 20px; outline: none; background: white; color: black; font-family: 'Times New Roman'; overflow-y: auto; }
        .terminal-area { background: #0c0c0c; color: #00f3ff; font-family: var(--mono); padding: 10px; height: 100%; overflow-y: auto; }
        .settings-area { padding: 20px; overflow-y: auto; height: 100%; }
        .setting-group { margin-bottom: 20px; border-bottom: 1px solid #333; padding-bottom: 10px; }
        .setting-label { display: block; margin-bottom: 5px; color: white; }
        .setting-input { width: 100%; padding: 8px; background: #111; border: 1px solid #444; color: white; border-radius: 4px; }
        .btn-action { padding: 8px 15px; background: var(--accent); color: black; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; margin-top: 5px; }

        /* --- TASKBAR --- */
        #taskbar {
            position: absolute; bottom: 0; width: 100%; height: 45px;
            background: rgba(15, 15, 20, 0.9); backdrop-filter: var(--blur);
            border-top: 1px solid #333; display: flex; align-items: center; justify-content: space-between; padding: 0 15px; z-index: 5000;
        }
        .start-btn { background: var(--accent); color: black; padding: 5px 15px; font-weight: bold; border-radius: 3px; cursor: pointer; }
        .tray { display: flex; gap: 15px; font-size: 12px; color: #aaa; align-items: center; }

        /* --- CONTEXT MENU --- */
        #context-menu {
            position: absolute; width: 180px; background: #2d2d2d; border: 1px solid #444;
            display: none; z-index: 9999; box-shadow: 0 5px 15px black;
        }
        .ctx-item { padding: 10px; cursor: pointer; font-size: 13px; color: white; }
        .ctx-item:hover { background: var(--accent); color: black; }

        /* Notifications */
        #notif-area { position: absolute; bottom: 60px; right: 20px; z-index: 9999; }
        .toast { background: #333; border-left: 3px solid var(--accent); color: white; padding: 10px 20px; margin-top: 10px; box-shadow: 0 5px 15px black; animation: slideIn 0.3s; }
        @keyframes slideIn { from { transform: translateX(100%); } to { transform: translateX(0); } }
    </style>
</head>
<body>

    <!-- LOCK SCREEN -->
    <div id="lock-screen">
        <div class="login-box">
            <div class="avatar"></div>
            <h2 id="lock-msg">SECURE LOGIN</h2>
            <input type="password" id="password-input" class="pass-input" placeholder="Enter Password">
        </div>
    </div>

    <!-- WALLPAPER -->
    <div id="wallpaper"></div>

    <!-- DESKTOP -->
    <div id="desktop"></div>

    <!-- WINDOWS CONTAINER -->
    <div id="windows-area"></div>

    <!-- TASKBAR -->
    <div id="taskbar">
        <div class="start-btn" onclick="toggleStart()">HYPER OS</div>
        <div class="tray">
            <span id="region-display">US/East</span>
            <span id="clock">12:00</span>
        </div>
    </div>

    <!-- CONTEXT MENU -->
    <div id="context-menu">
        <div class="ctx-item" onclick="Apps.createFile('doc')">📄 New Document</div>
        <div class="ctx-item" onclick="Apps.createFile('secret')">🔒 New Secret File</div>
        <div class="ctx-item" onclick="System.openApp('terminal', null)">💻 Open Terminal</div>
        <div class="ctx-item" onclick="System.openApp('settings', null)">⚙️ Settings</div>
    </div>

    <!-- NOTIFICATIONS -->
    <div id="notif-area"></div>

    <script>
        /* --- CORE SYSTEM & STORAGE --- */
        const System = {
            // Default Config
            defaultConfig: {
                password: "1234",
                wallpaper: "https://images.unsplash.com/photo-1451187580459-43490279c0fa?q=80&w=2072",
                region: "US/Washington",
                showSecrets: false
            },
            // Default Files
            defaultFiles: [
                { id: 'f1', name: 'Welcome.txt', type: 'doc', content: '<h1>Welcome to Hyper OS</h1><p>This system uses localStorage.</p><p>You can create files, save them, and refresh the page. They will persist!</p>', date: Date.now() },
                { id: 'f2', name: 'Codes.txt', type: 'doc', content: 'Top Secret Launch Codes: 000-000', isSecret: true, date: Date.now() }
            ],

            state: {},
            files: [],

            init: function() {
                // Load from LocalStorage
                const savedConfig = localStorage.getItem('hyper_config');
                const savedFiles = localStorage.getItem('hyper_files');

                this.state = savedConfig ? JSON.parse(savedConfig) : this.defaultConfig;
                this.files = savedFiles ? JSON.parse(savedFiles) : this.defaultFiles;

                this.applyConfig();
                this.renderDesktop();
                this.updateClock();
                setInterval(() => this.updateClock(), 1000);

                // Lock Screen Event
                document.getElementById('password-input').addEventListener('keyup', (e) => {
                    if(e.key === 'Enter') this.login();
                });
            },

            saveState: function() {
                localStorage.setItem('hyper_config', JSON.stringify(this.state));
                localStorage.setItem('hyper_files', JSON.stringify(this.files));
            },

            applyConfig: function() {
                document.getElementById('wallpaper').style.backgroundImage = `url('${this.state.wallpaper}')`;
                document.getElementById('region-display').innerText = this.state.region;
            },

            login: function() {
                const input = document.getElementById('password-input');
                if(input.value === this.state.password) {
                    const lock = document.getElementById('lock-screen');
                    lock.style.opacity = '0';
                    setTimeout(() => lock.style.display = 'none', 500);
                    this.notify("System Unlocked", "Welcome back, Agent.");
                } else {
                    input.style.border = "1px solid red";
                    input.value = "";
                }
            },

            notify: function(title, msg) {
                const area = document.getElementById('notif-area');
                const toast = document.createElement('div');
                toast.className = 'toast';
                toast.innerHTML = `<b>${title}</b><br>${msg}`;
                area.appendChild(toast);
                setTimeout(() => toast.remove(), 3000);
            },

            renderDesktop: function() {
                const desk = document.getElementById('desktop');
                desk.innerHTML = '';
                
                // Static Apps
                this.createIcon(desk, 'My Computer', 'pc', () => alert('System Info: v4.0 Persistent'));
                this.createIcon(desk, 'Recycle Bin', 'trash', () => alert('Bin is empty'));
                this.createIcon(desk, 'Settings', 'settings', () => this.openApp('settings', null));
                this.createIcon(desk, 'Terminal', 'terminal', () => this.openApp('terminal', null));

                // Dynamic Files
                this.files.forEach(file => {
                    if(file.isSecret && !this.state.showSecrets) return; // Hide secret files logic

                    let iconType = file.type === 'doc' ? 'doc' : 'file';
                    if(file.isSecret) iconType = 'secret';

                    this.createIcon(desk, file.name, iconType, () => this.openApp('editor', file.id));
                });
            },

            createIcon: function(container, name, type, action) {
                let img = '';
                if(type === 'pc') img = 'https://cdn-icons-png.flaticon.com/512/3067/3067469.png';
                else if(type === 'trash') img = 'https://cdn-icons-png.flaticon.com/512/3221/3221897.png';
                else if(type === 'settings') img = 'https://cdn-icons-png.flaticon.com/512/3953/3953226.png';
                else if(type === 'terminal') img = 'https://cdn-icons-png.flaticon.com/512/2620/2620958.png';
                else if(type === 'doc') img = 'https://cdn-icons-png.flaticon.com/512/5968/5968517.png';
                else if(type === 'secret') img = 'https://cdn-icons-png.flaticon.com/512/2965/2965302.png';

                const div = document.createElement('div');
                div.className = 'icon';
                div.innerHTML = `<img src="${img}"><span>${name}</span>`;
                div.onclick = action;
                div.oncontextmenu = (e) => {
                    e.preventDefault();
                    if(type === 'doc' || type === 'secret') {
                        if(confirm(`Delete ${name}?`)) {
                            this.files = this.files.filter(f => f.name !== name);
                            this.saveState();
                            this.renderDesktop();
                        }
                    }
                };
                container.appendChild(div);
            },

            updateClock: function() {
                const now = new Date();
                document.getElementById('clock').innerText = now.toLocaleTimeString();
            },

            /* --- APP WINDOW LOGIC --- */
            zIndex: 100,
            openApp: function(appType, fileId) {
                const win = document.createElement('div');
                win.className = 'window';
                win.style.zIndex = ++this.zIndex;
                
                // Content Switch
                let title = '';
                let contentHTML = '';

                if(appType === 'settings') {
                    title = 'System Settings';
                    contentHTML = `
                        <div class="settings-area">
                            <h2>Control Panel</h2>
                            <div class="setting-group">
                                <label class="setting-label">Wallpaper URL</label>
                                <input id="set-bg" class="setting-input" value="${this.state.wallpaper}">
                                <button class="btn-action" onclick="Apps.saveSettings('bg')">Apply</button>
                            </div>
                            <div class="setting-group">
                                <label class="setting-label">System Password</label>
                                <input id="set-pass" class="setting-input" value="${this.state.password}">
                                <button class="btn-action" onclick="Apps.saveSettings('pass')">Update</button>
                            </div>
                            <div class="setting-group">
                                <label class="setting-label">Region (Text Display)</label>
                                <select id="set-reg" class="setting-input">
                                    <option value="US/Washington">US/Washington</option>
                                    <option value="EU/London">EU/London</option>
                                    <option value="RU/Moscow">RU/Moscow</option>
                                </select>
                                <button class="btn-action" onclick="Apps.saveSettings('reg')">Set Region</button>
                            </div>
                            <div class="setting-group" style="border:none;">
                                <label class="setting-label" style="color:red;">Red Protocol (Show Secrets)</label>
                                <button class="btn-action" style="background:${this.state.showSecrets ? 'red':'#333'}; color:white;" onclick="Apps.toggleSecrets()">
                                    ${this.state.showSecrets ? 'DEACTIVATE' : 'ACTIVATE'}
                                </button>
                            </div>
                        </div>
                    `;
                } else if (appType === 'editor') {
                    const file = this.files.find(f => f.id === fileId);
                    title = file.name;
                    contentHTML = `
                        <div style="display:flex; flex-direction:column; height:100%;">
                            <div style="background:#f0f0f0; padding:5px;">
                                <button onclick="Apps.saveDoc('${fileId}', this)">💾 SAVE</button>
                            </div>
                            <div class="editor-area" id="editor-${fileId}" contenteditable="true">${file.content}</div>
                        </div>
                    `;
                } else if (appType === 'terminal') {
                    title = 'Bash Terminal';
                    contentHTML = `
                        <div class="terminal-area" id="term-out" onclick="document.getElementById('term-in').focus()">
                            <div>Hyper Linux 4.0 [Persistent Core]</div>
                            <div>Type 'help' for commands.</div><br>
                            <div id="term-history"></div>
                            <div style="display:flex;">
                                <span>root@hyper:~#&nbsp;</span>
                                <input id="term-in" style="background:none; border:none; color:white; font-family:inherit; flex:1; outline:none;" autocomplete="off">
                            </div>
                        </div>
                    `;
                    setTimeout(Apps.initTerminal, 100);
                }

                win.innerHTML = `
                    <div class="titlebar" onmousedown="System.dragWindow(event, this.parentNode)">
                        <span>${title}</span>
                        <div class="win-controls">
                            <button class="btn-min" onclick="this.closest('.window').style.display='none'"></button>
                            <button class="btn-close" onclick="this.closest('.window').remove()"></button>
                        </div>
                    </div>
                    <div class="win-content">${contentHTML}</div>
                `;

                document.getElementById('windows-area').appendChild(win);
                setTimeout(() => win.classList.add('open'), 10);
                win.onmousedown = () => win.style.zIndex = ++this.zIndex;
            },

            dragWindow: function(e, win) {
                win.style.zIndex = ++this.zIndex;
                let shiftX = e.clientX - win.getBoundingClientRect().left;
                let shiftY = e.clientY - win.getBoundingClientRect().top;
                
                function moveAt(pageX, pageY) {
                    win.style.left = pageX - shiftX + 'px';
                    win.style.top = pageY - shiftY + 'px';
                }
                function onMouseMove(event) { moveAt(event.pageX, event.pageY); }
                document.addEventListener('mousemove', onMouseMove);
                win.onmouseup = function() {
                    document.removeEventListener('mousemove', onMouseMove);
                    win.onmouseup = null;
                };
            }
        };

        /* --- APPLICATIONS LOGIC --- */
        const Apps = {
            createFile: function(type) {
                const name = prompt("Enter file name:", type === 'doc' ? "New Doc.txt" : "Secret.txt");
                if(!name) return;
                
                const newFile = {
                    id: 'f-' + Date.now(),
                    name: name,
                    type: type,
                    content: '<h1>New File</h1><p>Start typing here...</p>',
                    isSecret: type === 'secret',
                    date: Date.now()
                };
                
                System.files.push(newFile);
                System.saveState();
                System.renderDesktop();
            },

            saveDoc: function(id, btn) {
                const content = document.getElementById('editor-' + id).innerHTML;
                const fileIndex = System.files.findIndex(f => f.id === id);
                if(fileIndex > -1) {
                    System.files[fileIndex].content = content;
                    System.saveState();
                    
                    // Button Feedback
                    const originalText = btn.innerText;
                    btn.innerText = "✅ SAVED";
                    setTimeout(() => btn.innerText = originalText, 1000);
                }
            },

            saveSettings: function(type) {
                if(type === 'bg') {
                    System.state.wallpaper = document.getElementById('set-bg').value;
                    System.applyConfig();
                } else if (type === 'pass') {
                    System.state.password = document.getElementById('set-pass').value;
                    alert("Password updated!");
                } else if (type === 'reg') {
                    System.state.region = document.getElementById('set-reg').value;
                    System.applyConfig();
                }
                System.saveState();
            },

            toggleSecrets: function() {
                const pass = prompt("Enter ADMIN Password to toggle protocol:");
                if(pass === System.state.password) {
                    System.state.showSecrets = !System.state.showSecrets;
                    System.saveState();
                    System.renderDesktop(); // Re-render to show/hide icons
                    // Close settings window to refresh view
                    document.querySelector('.window .settings-area').closest('.window').remove();
                    System.openApp('settings', null);
                } else {
                    alert("ACCESS DENIED");
                }
            },

            initTerminal: function() {
                const input = document.getElementById('term-in');
                const history = document.getElementById('term-history');
                
                input.focus();
                input.addEventListener('keydown', (e) => {
                    if(e.key === 'Enter') {
                        const cmd = input.value.trim();
                        const line = document.createElement('div');
                        line.innerHTML = `root@hyper:~# ${cmd}`;
                        history.appendChild(line);

                        const resp = document.createElement('div');
                        resp.style.color = "#ccc";
                        resp.style.marginBottom = "5px";

                        // COMMANDS
                        if(cmd === 'help') {
                            resp.innerHTML = "Commands: <br> - ls (list files)<br> - date<br> - whoami<br> - clear<br> - reboot (reset local data)";
                        } else if (cmd === 'ls') {
                            let fileList = System.files.map(f => `${f.isSecret ? '[LOCKED] ' : ''}${f.name}`).join('<br>');
                            resp.innerHTML = fileList || "No files found.";
                        } else if (cmd === 'date') {
                            resp.innerText = new Date().toString();
                        } else if (cmd === 'whoami') {
                            resp.innerText = "root (Administrator)";
                        } else if (cmd === 'reboot') {
                            if(confirm("Factory Reset? All data will be lost.")) {
                                localStorage.clear();
                                location.reload();
                            }
                        } else if (cmd === 'clear') {
                            history.innerHTML = '';
                            resp.remove();
                        } else {
                            resp.innerText = `Command not found: ${cmd}`;
                        }
                        
                        if(cmd !== 'clear') history.appendChild(resp);
                        input.value = '';
                        document.getElementById('term-out').scrollTop = document.getElementById('term-out').scrollHeight;
                    }
                });
            }
        };

        /* --- GLOBAL EVENTS --- */
        document.addEventListener('contextmenu', (e) => {
            e.preventDefault();
            const ctx = document.getElementById('context-menu');
            ctx.style.display = 'block';
            ctx.style.left = e.pageX + 'px';
            ctx.style.top = e.pageY + 'px';
        });

        document.addEventListener('click', (e) => {
            document.getElementById('context-menu').style.display = 'none';
        });

        // START SYSTEM
        System.init();

    </script>
</body>
</html>
