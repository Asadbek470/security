# security
.
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hyper OS: Ultimate Secure Edition</title>
    <style>
        /* --- THEME VARIABLES --- */
        :root {
            --bg-color: #050505;
            --window-bg: #1e1e1e;
            --text-color: #ffffff;
            --accent: #00f3ff; /* Default Blue */
            --accent-sec: #0066ff;
            --danger: #ff2a2a;
            --font-main: 'Segoe UI', Roboto, Helvetica, sans-serif;
            --font-mono: 'Consolas', 'Courier New', monospace;
            --glass: rgba(20, 25, 35, 0.85);
            --blur: blur(15px);
            --border: 1px solid rgba(255, 255, 255, 0.1);
            --taskbar-h: 45px;
        }

        /* THEMES */
        [data-theme="cyberpunk"] {
            --bg-color: #0b0014;
            --window-bg: #1a0518;
            --accent: #ff0055; /* Neon Pink */
            --accent-sec: #fcee0a; /* Yellow */
            --font-main: 'Segoe UI', sans-serif;
            --border: 1px solid #ff0055;
            --glass: rgba(40, 0, 40, 0.9);
        }

        [data-theme="hacker"] {
            --bg-color: #000000;
            --window-bg: #0d0d0d;
            --accent: #00ff00; /* Matrix Green */
            --accent-sec: #008f11;
            --text-color: #00ff00;
            --font-main: 'Courier New', monospace;
            --border: 1px solid #00ff00;
            --glass: rgba(0, 20, 0, 0.95);
        }

        * { box-sizing: border-box; user-select: none; }
        body { margin: 0; overflow: hidden; background: #000; font-family: var(--font-main); color: var(--text-color); height: 100vh; }

        /* --- WALLPAPER & LAYOUT --- */
        #wallpaper {
            position: absolute; inset: 0; z-index: 0;
            background-size: cover; background-position: center;
            transition: 0.5s;
            filter: brightness(0.6);
        }

        /* --- LOCK SCREEN --- */
        #lock-screen {
            position: fixed; inset: 0; z-index: 10000;
            background: black;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            transition: opacity 0.5s;
        }
        .login-box { text-align: center; width: 350px; padding: 40px; border: var(--border); background: rgba(0,0,0,0.8); box-shadow: 0 0 30px var(--accent); }
        .avatar { width: 100px; height: 100px; border-radius: 50%; margin: 0 auto 20px; border: 3px solid var(--accent); background-image: url('https://cdn-icons-png.flaticon.com/512/3135/3135715.png'); background-size: cover; }
        .pass-input {
            width: 100%; padding: 12px; background: rgba(255,255,255,0.05); border: 1px solid #444;
            color: var(--text-color); text-align: center; font-size: 18px; border-radius: 4px; outline: none; margin-top: 15px; font-family: var(--font-mono);
        }
        .pass-input:focus { border-color: var(--accent); box-shadow: 0 0 10px var(--accent); }
        
        .error-shake { animation: shake 0.3s; border-color: var(--danger) !important; }
        @keyframes shake { 0% { transform: translateX(0); } 25% { transform: translateX(-10px); } 75% { transform: translateX(10px); } 100% { transform: translateX(0); } }

        /* --- DESKTOP --- */
        #desktop { position: absolute; inset: 0; z-index: 10; padding: 20px; display: flex; flex-direction: column; flex-wrap: wrap; align-content: flex-start; gap: 15px; height: calc(100vh - var(--taskbar-h)); }

        .icon {
            width: 90px; height: 100px; display: flex; flex-direction: column; align-items: center;
            cursor: pointer; border-radius: 5px; text-align: center; transition: 0.2s;
            padding: 5px;
        }
        .icon:hover { background: rgba(255,255,255,0.1); transform: translateY(-2px); }
        .icon img { width: 50px; height: 50px; margin-bottom: 8px; filter: drop-shadow(0 4px 5px rgba(0,0,0,0.8)); }
        .icon span { font-size: 13px; text-shadow: 0 2px 4px black; line-height: 1.2; font-weight: 500; }

        /* --- WINDOWS --- */
        .window {
            position: absolute; width: 600px; height: 450px;
            background: var(--window-bg); border: var(--border); border-radius: 6px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.9);
            display: flex; flex-direction: column; overflow: hidden;
            top: 10%; left: 15%; opacity: 0; transform: scale(0.9);
            transition: opacity 0.2s, transform 0.2s;
            resize: both;
        }
        .window.open { opacity: 1; transform: scale(1); }

        .titlebar {
            height: 35px; background: rgba(255,255,255,0.05); display: flex; align-items: center; justify-content: space-between; padding: 0 10px; cursor: grab; border-bottom: 1px solid rgba(255,255,255,0.1);
        }
        .titlebar span { font-size: 14px; font-weight: bold; letter-spacing: 1px; color: var(--accent); }
        .win-controls button { width: 14px; height: 14px; border-radius: 50%; border: none; margin-left: 6px; cursor: pointer; }
        .btn-close { background: #ff5555; } .btn-min { background: #ffbd44; } .btn-max { background: #00ca4e; }

        .win-content { flex: 1; background: rgba(0,0,0,0.2); overflow: hidden; position: relative; display: flex; flex-direction: column; }
        
        /* App Layouts */
        .browser-bar { display: flex; gap: 5px; padding: 8px; background: #222; border-bottom: 1px solid #333; }
        .url-input { flex: 1; padding: 5px 10px; border-radius: 20px; border: none; background: #111; color: white; outline: none; }
        .browser-frame { flex: 1; background: white; border: none; }
        .browser-placeholder { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; color: #aaa; text-align: center; padding: 20px; }
        
        .editor-area { width: 100%; height: 100%; padding: 20px; outline: none; background: #fff; color: #000; overflow-y: auto; font-family: 'Times New Roman'; }
        [data-theme="hacker"] .editor-area { background: #000; color: #0f0; font-family: monospace; }
        
        .image-viewer { width: 100%; height: 100%; display: flex; align-items: center; justify-content: center; background: #111; }
        .image-viewer img { max-width: 100%; max-height: 100%; object-fit: contain; }

        .settings-area { padding: 25px; overflow-y: auto; height: 100%; }
        .setting-group { margin-bottom: 25px; border-bottom: 1px solid #333; padding-bottom: 15px; }
        .setting-label { display: block; margin-bottom: 8px; color: var(--accent); font-weight: bold; }
        .setting-input { width: 100%; padding: 10px; background: #111; border: 1px solid #444; color: white; border-radius: 4px; outline: none; }
        .btn-action { padding: 8px 20px; background: var(--accent); color: black; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; margin-top: 10px; transition: 0.2s; }
        .btn-action:hover { filter: brightness(1.2); }

        /* --- TASKBAR --- */
        #taskbar {
            position: absolute; bottom: 0; width: 100%; height: var(--taskbar-h);
            background: var(--glass); backdrop-filter: var(--blur);
            border-top: var(--border); display: flex; align-items: center; justify-content: space-between; padding: 0 15px; z-index: 5000;
        }
        .start-btn { 
            background: linear-gradient(45deg, var(--accent), var(--accent-sec)); 
            color: black; padding: 6px 18px; font-weight: bold; border-radius: 4px; cursor: pointer; 
            box-shadow: 0 0 10px var(--accent); text-transform: uppercase; letter-spacing: 1px;
        }
        .tray { display: flex; gap: 15px; font-size: 13px; color: #ccc; align-items: center; font-family: var(--font-mono); }

        /* --- CONTEXT MENU --- */
        #context-menu {
            position: absolute; width: 220px; background: #222; border: 1px solid #444;
            display: none; z-index: 9999; box-shadow: 5px 5px 15px black;
        }
        .ctx-item { padding: 12px; cursor: pointer; font-size: 13px; color: white; border-bottom: 1px solid #333; }
        .ctx-item:hover { background: var(--accent); color: black; }
        .ctx-item:last-child { border-bottom: none; }

        /* Notifications */
        #notif-area { position: absolute; bottom: 60px; right: 20px; z-index: 9999; display: flex; flex-direction: column; gap: 10px; }
        .toast { 
            background: rgba(0,0,0,0.9); border-right: 4px solid var(--accent); color: white; 
            padding: 15px 25px; min-width: 250px; box-shadow: -5px 5px 15px black; 
            animation: slideIn 0.3s forwards; opacity: 0; transform: translateX(100%);
        }
        @keyframes slideIn { to { opacity: 1; transform: translateX(0); } }

        iframe { border: none; width: 100%; height: 100%; }
    </style>
</head>
<body data-theme="default">

    <!-- LOCK SCREEN -->
    <div id="lock-screen">
        <div class="login-box">
            <div class="avatar"></div>
            <h2 id="lock-msg" style="color:var(--accent); text-transform:uppercase;">Система Безопасности</h2>
            <input type="password" id="password-input" class="pass-input" placeholder="Введите пароль">
            <p style="font-size:12px; color:#666; margin-top:10px;">Подсказка по умолчанию: 1234</p>
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
        <div class="start-btn" onclick="Apps.toggleMenu()">ПУСК</div>
        <div class="tray">
            <span id="security-status" style="color:var(--accent);">● ЗАЩИЩЕНО</span>
            <span id="clock">12:00</span>
        </div>
    </div>

    <!-- CONTEXT MENU -->
    <div id="context-menu">
        <div class="ctx-item" onclick="Apps.createFile('doc')">📄 Новый Документ</div>
        <div class="ctx-item" onclick="Apps.createFile('secret')">🔒 Секретный файл (Текст)</div>
        <div class="ctx-item" onclick="Apps.createFile('secret-img')">🖼️ Секретная Картинка (Маскировка)</div>
        <div class="ctx-item" onclick="System.openApp('browser', null)">🌐 Браузер Google</div>
        <div class="ctx-item" onclick="System.openApp('settings', null)">⚙️ Настройки</div>
    </div>

    <!-- NOTIFICATIONS -->
    <div id="notif-area"></div>

    <script>
        /* --- CORE SYSTEM & STORAGE --- */
        const System = {
            defaultConfig: {
                password: "1234",
                wallpaper: "https://images.unsplash.com/photo-1550751827-4bd374c3f58b?q=80&w=2070", // Cyberpunk city
                theme: "default"
            },
            defaultFiles: [
                { id: 'f1', name: 'О Системе.txt', type: 'doc', content: '<h1>Hyper OS v5.0</h1><p>Система обновлена. Добавлена поддержка Google, YouTube и маскировка файлов.</p>', date: Date.now() },
            ],

            state: {},
            files: [],
            clipboard: null,

            init: function() {
                const savedConfig = localStorage.getItem('hyper_config_v2');
                const savedFiles = localStorage.getItem('hyper_files_v2');

                this.state = savedConfig ? JSON.parse(savedConfig) : this.defaultConfig;
                this.files = savedFiles ? JSON.parse(savedFiles) : this.defaultFiles;

                this.applyConfig();
                this.renderDesktop();
                this.updateClock();
                setInterval(() => this.updateClock(), 1000);

                // Lock Screen Input
                const passInput = document.getElementById('password-input');
                passInput.focus();
                passInput.addEventListener('keyup', (e) => {
                    if(e.key === 'Enter') this.login();
                });
            },

            saveState: function() {
                localStorage.setItem('hyper_config_v2', JSON.stringify(this.state));
                localStorage.setItem('hyper_files_v2', JSON.stringify(this.files));
            },

            applyConfig: function() {
                document.getElementById('wallpaper').style.backgroundImage = `url('${this.state.wallpaper}')`;
                document.body.setAttribute('data-theme', this.state.theme);
            },

            login: function() {
                const input = document.getElementById('password-input');
                if(input.value === this.state.password) {
                    const lock = document.getElementById('lock-screen');
                    lock.style.opacity = '0';
                    setTimeout(() => lock.style.display = 'none', 500);
                    this.notify("Доступ разрешен", "Добро пожаловать в систему.");
                    // Play startup sound simulated
                } else {
                    input.classList.add('error-shake');
                    input.value = "";
                    input.placeholder = "НЕВЕРНЫЙ ПАРОЛЬ";
                    setTimeout(() => input.classList.remove('error-shake'), 500);
                }
            },

            notify: function(title, msg) {
                const area = document.getElementById('notif-area');
                const toast = document.createElement('div');
                toast.className = 'toast';
                toast.innerHTML = `<b>${title}</b><br>${msg}`;
                area.appendChild(toast);
                setTimeout(() => toast.remove(), 4000);
            },

            renderDesktop: function() {
                const desk = document.getElementById('desktop');
                desk.innerHTML = '';
                
                // System Apps
                this.createIcon(desk, 'Мой ПК', 'pc', () => alert('Процессор: Quantum Core i9\nПамять: 128 TB\nЗащита: Активна'));
                this.createIcon(desk, 'Google', 'chrome', () => this.openApp('browser', null));
                this.createIcon(desk, 'YouTube', 'youtube', () => this.openApp('youtube', null));
                this.createIcon(desk, 'WhatsApp', 'whatsapp', () => this.openLink('https://web.whatsapp.com'));
                this.createIcon(desk, 'Telegram', 'telegram', () => this.openLink('https://web.telegram.org'));
                this.createIcon(desk, 'Настройки', 'settings', () => this.openApp('settings', null));
                this.createIcon(desk, 'Корзина', 'trash', () => alert('Корзина пуста'));

                // User Files
                this.files.forEach(file => {
                    let iconType = 'file';
                    // Маскировка: Секретные картинки выглядят как документы
                    if(file.type === 'doc') iconType = 'doc';
                    else if(file.type === 'secret') iconType = 'lock';
                    else if(file.type === 'secret-img') iconType = 'doc'; // Disguise!

                    this.createIcon(desk, file.name, iconType, () => this.handleFileOpen(file));
                });
            },

            createIcon: function(container, name, type, action) {
                const icons = {
                    pc: 'https://cdn-icons-png.flaticon.com/512/3067/3067469.png',
                    chrome: 'https://cdn-icons-png.flaticon.com/512/2991/2991148.png',
                    youtube: 'https://cdn-icons-png.flaticon.com/512/1384/1384060.png',
                    whatsapp: 'https://cdn-icons-png.flaticon.com/512/733/733585.png',
                    telegram: 'https://cdn-icons-png.flaticon.com/512/2111/2111646.png',
                    settings: 'https://cdn-icons-png.flaticon.com/512/3953/3953226.png',
                    trash: 'https://cdn-icons-png.flaticon.com/512/3221/3221897.png',
                    doc: 'https://cdn-icons-png.flaticon.com/512/5968/5968517.png',
                    lock: 'https://cdn-icons-png.flaticon.com/512/2965/2965302.png',
                    file: 'https://cdn-icons-png.flaticon.com/512/2965/2965302.png'
                };

                const div = document.createElement('div');
                div.className = 'icon';
                div.innerHTML = `<img src="${icons[type] || icons.file}"><span>${name}</span>`;
                div.onclick = action;
                div.oncontextmenu = (e) => {
                    e.preventDefault();
                    if(confirm(`Удалить ${name}?`)) {
                        this.files = this.files.filter(f => f.name !== name);
                        this.saveState();
                        this.renderDesktop();
                    }
                };
                container.appendChild(div);
            },

            openLink: function(url) {
                window.open(url, '_blank');
            },

            handleFileOpen: function(file) {
                if(file.type.startsWith('secret')) {
                    const pass = prompt("🔐 Введите пароль для доступа к файлу:");
                    if(pass === this.state.password) {
                        if(file.type === 'secret-img') {
                            this.openApp('image-view', file);
                        } else {
                            this.openApp('editor', file);
                        }
                    } else {
                        this.notify("ОШИБКА", "Неверный пароль! Попытка взлома зафиксирована.");
                    }
                } else {
                    this.openApp('editor', file);
                }
            },

            updateClock: function() {
                const now = new Date();
                document.getElementById('clock').innerText = now.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
            },

            /* --- APP WINDOW SYSTEM --- */
            zIndex: 100,
            openApp: function(appType, data) {
                const win = document.createElement('div');
                win.className = 'window';
                win.style.zIndex = ++this.zIndex;
                
                let title = 'Приложение';
                let contentHTML = '';

                if(appType === 'settings') {
                    title = 'Настройки Системы';
                    contentHTML = `
                        <div class="settings-area">
                            <div class="setting-group">
                                <label class="setting-label">Тема Интерфейса</label>
                                <select id="set-theme" class="setting-input" onchange="Apps.saveSettings('theme')">
                                    <option value="default" ${this.state.theme === 'default' ? 'selected' : ''}>Hyper Default (Blue)</option>
                                    <option value="cyberpunk" ${this.state.theme === 'cyberpunk' ? 'selected' : ''}>Cyberpunk 2077</option>
                                    <option value="hacker" ${this.state.theme === 'hacker' ? 'selected' : ''}>Matrix Hacker</option>
                                </select>
                            </div>
                            <div class="setting-group">
                                <label class="setting-label">URL Обоев</label>
                                <input id="set-bg" class="setting-input" value="${this.state.wallpaper}">
                                <button class="btn-action" onclick="Apps.saveSettings('bg')">Применить</button>
                            </div>
                            <div class="setting-group">
                                <label class="setting-label">Пароль Системы</label>
                                <input id="set-pass" class="setting-input" type="password" value="${this.state.password}">
                                <button class="btn-action" onclick="Apps.saveSettings('pass')">Обновить защиту</button>
                            </div>
                        </div>
                    `;
                } else if (appType === 'editor') {
                    title = data.name + (data.type === 'secret' ? ' [РАССЕКРЕЧЕНО]' : '');
                    contentHTML = `
                        <div style="display:flex; flex-direction:column; height:100%;">
                            <div style="background:#222; padding:5px; display:flex; gap:5px;">
                                <button onclick="Apps.saveDoc('${data.id}', this)" class="btn-action" style="margin:0; padding:4px 10px;">💾 Сохранить</button>
                                ${data.type !== 'secret' ? `<button onclick="Apps.insertImage('${data.id}')" class="btn-action" style="margin:0; padding:4px 10px;">🖼 Вставить фото</button>` : ''}
                            </div>
                            <div class="editor-area" id="editor-${data.id}" contenteditable="true">${data.content}</div>
                        </div>
                    `;
                } else if (appType === 'image-view') {
                    title = 'Просмотр: ' + data.name;
                    contentHTML = `
                        <div class="image-viewer">
                            <img src="${data.content}" alt="Secret Image" onerror="this.src='https://via.placeholder.com/400?text=Error+Loading+Image'">
                        </div>
                    `;
                } else if (appType === 'browser') {
                    title = 'Google Chrome (Secure Proxy)';
                    contentHTML = `
                        <div style="display:flex; flex-direction:column; height:100%;">
                            <div class="browser-bar">
                                <span style="font-size:20px; cursor:pointer;" onclick="Apps.navBrowser('back')">⬅</span>
                                <input id="browser-url" class="url-input" placeholder="Введите запрос или URL картинки" value="google.com">
                                <button class="btn-action" style="margin:0;" onclick="Apps.navBrowser('go')">GO</button>
                            </div>
                            <div class="browser-placeholder" id="browser-content">
                                <img src="https://www.google.com/images/branding/googlelogo/1x/googlelogo_light_color_272x92dp.png" width="200" style="opacity:0.8"><br><br>
                                <h2>Безопасный Поиск</h2>
                                <p>Введите запрос сверху. Поиск Google откроется в защищенной вкладке.</p>
                                <p><i>Чтобы скачать картинку: Найдите её в Google -> ПКМ -> "Копировать URL картинки" -> Вставьте в секретный файл.</i></p>
                            </div>
                        </div>
                    `;
                } else if (appType === 'youtube') {
                    title = 'YouTube';
                    win.style.width = '700px';
                    contentHTML = `
                        <div style="display:flex; flex-direction:column; height:100%;">
                            <div class="browser-bar">
                                <input id="yt-search" class="url-input" placeholder="Поиск видео...">
                                <button class="btn-action" style="margin:0;" onclick="Apps.searchYT()">Искать</button>
                            </div>
                             <iframe id="yt-frame" src="https://www.youtube.com/embed/dQw4w9WgXcQ" allowfullscreen></iframe>
                        </div>
                    `;
                }

                win.innerHTML = `
                    <div class="titlebar" onmousedown="System.dragWindow(event, this.parentNode)">
                        <span>${title}</span>
                        <div class="win-controls">
                            <button class="btn-min" onclick="this.closest('.window').style.display='none'"></button>
                            <button class="btn-max" onclick="this.closest('.window').classList.toggle('maximized')"></button>
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

        /* --- APP LOGIC --- */
        const Apps = {
            createFile: function(type) {
                let name = "";
                let content = "";
                let fileType = type;

                if (type === 'secret-img') {
                    name = prompt("Имя скрытого файла (будет выглядеть как документ):", "Project_X.txt");
                    const url = prompt("Вставьте прямую ссылку на картинку (URL):", "");
                    if (!url) return;
                    content = url;
                } else {
                    name = prompt("Имя файла:", type === 'secret' ? "Secret_Pass.txt" : "Новый документ.txt");
                    content = '<h1>Новый файл</h1>';
                }

                if(!name) return;
                
                const newFile = {
                    id: 'f-' + Date.now(),
                    name: name,
                    type: fileType,
                    content: content,
                    date: Date.now()
                };
                
                System.files.push(newFile);
                System.saveState();
                System.renderDesktop();
            },

            saveDoc: function(id, btn) {
                const editor = document.getElementById('editor-' + id);
                if(!editor) return;
                
                const fileIndex = System.files.findIndex(f => f.id === id);
                if(fileIndex > -1) {
                    System.files[fileIndex].content = editor.innerHTML;
                    System.saveState();
                    
                    const originalText = btn.innerText;
                    btn.innerText = "✅ ОК";
                    setTimeout(() => btn.innerText = originalText, 1000);
                }
            },

            insertImage: function(id) {
                const url = prompt("Введите URL картинки:");
                if(url) {
                    const imgTag = `<img src="${url}" style="max-width:100%;">`;
                    document.getElementById('editor-' + id).innerHTML += `<br>${imgTag}<br>`;
                }
            },

            saveSettings: function(key) {
                if(key === 'bg') System.state.wallpaper = document.getElementById('set-bg').value;
                if(key === 'pass') {
                     System.state.password = document.getElementById('set-pass').value;
                     alert('Пароль обновлен');
                }
                if(key === 'theme') System.state.theme = document.getElementById('set-theme').value;
                
                System.applyConfig();
                System.saveState();
            },

            navBrowser: function(action) {
                const input = document.getElementById('browser-url');
                const content = document.getElementById('browser-content');
                let val = input.value;

                if(action === 'go') {
                    if(val.includes('http') && (val.endsWith('.jpg') || val.endsWith('.png'))) {
                        // Это картинка - показать внутри
                        content.innerHTML = `<img src="${val}" style="max-width:100%; max-height:100%;">`;
                    } else {
                        // Это поиск
                        window.open(`https://www.google.com/search?q=${val}`, '_blank');
                        content.innerHTML = `<h3 style="color:var(--accent)">Запрос отправлен в новую вкладку</h3><p>Google блокирует открытие внутри фреймов.</p>`;
                    }
                }
            },

            searchYT: function() {
                const query = document.getElementById('yt-search').value;
                if(query) {
                    window.open(`https://www.youtube.com/results?search_query=${query}`, '_blank');
                }
            },

            toggleMenu: function() {
               // Placeholder for start menu logic if needed later
               const ctx = document.getElementById('context-menu');
               ctx.style.display = 'block';
               ctx.style.bottom = '50px';
               ctx.style.left = '10px';
               ctx.style.top = 'auto';
            }
        };

        /* --- GLOBAL EVENTS --- */
        document.addEventListener('contextmenu', (e) => {
            if(e.target.closest('.window')) return; // Allow context menu in windows? No, keep it custom
            e.preventDefault();
            const ctx = document.getElementById('context-menu');
            ctx.style.display = 'block';
            ctx.style.left = e.pageX + 'px';
            ctx.style.top = e.pageY + 'px';
            ctx.style.bottom = 'auto';
        });

        document.addEventListener('click', (e) => {
            document.getElementById('context-menu').style.display = 'none';
        });

        // INIT
        System.init();

    </script>
</body>
</html>
