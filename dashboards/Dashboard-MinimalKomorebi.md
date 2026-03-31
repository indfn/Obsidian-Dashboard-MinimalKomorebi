---
cssclasses:
  - dashboard
  - max
obsidianUIMode: preview
---
```dataviewjs
// Dashboard-Komorebi
// more dashboards → https://github.com/InlitX/Obsidian-Dashboard-Gallery

const wrap = dv.container.createDiv({ cls: 'komo-header-block' });
const hdr  = wrap.createDiv({ cls: 'komo-header' });

// ── BRAND (left) ─────────────────────────────
const brand = hdr.createDiv({ cls: 'komo-brand' });

const savedTitle  = localStorage.getItem('komo-title')  || 'KOMOREBI';
const savedMantra = localStorage.getItem('komo-mantra') || 'notes, thoughts & things that matter';

const titleEl = brand.createEl('div', {
    cls: 'komo-title',
    attr: { contenteditable: 'true', spellcheck: 'false', 'data-placeholder': 'KOMOREBI' }
});
titleEl.textContent = savedTitle;
titleEl.addEventListener('blur', () => {
    const v = titleEl.textContent.trim();
    if (v) localStorage.setItem('komo-title', v);
});

const mantraEl = brand.createEl('div', {
    cls: 'komo-mantra',
    attr: { contenteditable: 'true', spellcheck: 'false' }
});
mantraEl.textContent = savedMantra;
mantraEl.addEventListener('blur', () => {
    const v = mantraEl.textContent.trim();
    if (v) localStorage.setItem('komo-mantra', v);
});

// ── CLOCK (center) ────────────────────────────
const clk    = hdr.createDiv({ cls: 'komo-clock-block' });
const timeEl = clk.createDiv({ cls: 'komo-time' });
const dateEl = clk.createDiv({ cls: 'komo-date' });

function tick() {
    const now = new Date();
    const pad = n => String(n).padStart(2, '0');
    timeEl.textContent = `${pad(now.getHours())}:${pad(now.getMinutes())}:${pad(now.getSeconds())}`;
    const longDate = now.toLocaleDateString('en-US', {
        weekday: 'long', year: 'numeric', month: 'long', day: 'numeric'
    });
    dateEl.textContent = longDate;
}
tick();
setInterval(tick, 1000);

// ── RIGHT — GREETING + STATS ──────────────────
const rightHdr = hdr.createDiv({ cls: 'komo-header-right' });

const HOUR = new Date().getHours();
const GREETS = [
    { h: [0,5],  period: 'Late Night',     en: 'Still going?' },
    { h: [5,9],  period: 'Early Morning',  en: 'New day, new flow.' },
    { h: [9,13], period: 'Morning',        en: 'Peak focus. Engage.' },
    { h: [13,18],period: 'Afternoon',      en: 'Stay in the flow.' },
    { h: [18,21],period: 'Evening',        en: 'Review your progress.' },
    { h: [21,24],period: 'Night',          en: 'Wind down. Reflect.' }
];
const G = GREETS.find(g => HOUR >= g.h[0] && HOUR < g.h[1]) || GREETS[5];
const gEl = rightHdr.createDiv({ cls: 'komo-greeting' });
gEl.createEl('span', { cls: 'komo-greet-jp', text: G.period });
gEl.createEl('span', { cls: 'komo-greet-en', text: G.en });

// Vault stats
const statsRow = rightHdr.createDiv({ cls: 'komo-stats-row' });
const pages  = dv.pages();
const tasks  = pages.file.tasks;
const doneN  = tasks.where(t => t.completed).length;
const openN  = tasks.where(t => !t.completed).length;

[
    { n: pages.length, l: 'notes' },
    { n: openN,        l: 'open'  },
    { n: doneN,        l: 'done'  },
].forEach(({ n, l }) => {
    const p = statsRow.createDiv({ cls: 'komo-pill' });
    p.createDiv({ cls: 'komo-pill-num', text: n.toString() });
    p.createDiv({ cls: 'komo-pill-lbl', text: l });
});

// Thin sakura divider
wrap.createDiv({ cls: 'komo-divider' });
```

```dataviewjs
// Dashboard-Komorebi
// more dashboards → https://github.com/InlitX/Obsidian-Dashboard-Gallery

const wrap  = dv.container.createDiv({ cls: 'komo-grid-block' });
const grid  = wrap.createDiv({ cls: 'komo-grid' });

// ══════════════════════════════════════════════
//  LEFT COLUMN
// ══════════════════════════════════════════════
const colL = grid.createDiv({ cls: 'komo-col-left' });

// ── FOCUS CARD ───────────────────────────────
const focusCard = colL.createDiv({ cls: 'komo-card' });
focusCard.createDiv({ cls: 'komo-label', text: "today's goal" });

const focusEl = focusCard.createEl('div', {
    cls: 'komo-focus',
    attr: { contenteditable: 'true', spellcheck: 'false', placeholder: 'define your focus...' }
});
focusEl.textContent = localStorage.getItem('komo-focus') || 'do the work. trust the process.';
focusEl.addEventListener('blur', () => {
    localStorage.setItem('komo-focus', focusEl.textContent.trim());
});

// ── SYSTEM CARD ──────────────────────────────
const sysCard = colL.createDiv({ cls: 'komo-card komo-sys-card' });
sysCard.createDiv({ cls: 'komo-label', text: 'system' });

const ACTIONS = [
    { icon: 'calendar',    lbl: 'daily',   em: '📅' },
    { icon: 'search',      lbl: 'search',  cmd: 'global-search:open',        em: '🔍' },
    { icon: 'share-2',     lbl: 'graph',   cmd: 'graph:open',                em: '🕸' },
    { icon: 'file-plus',   lbl: 'new',     cmd: 'file-explorer:new-file',    em: '＋' },
    { icon: 'zap',         lbl: 'quick',   cmd: 'quickadd:runQuickAdd',      em: '⚡' },
    { icon: 'terminal',    lbl: 'cmds',    cmd: 'command-palette:open',      em: '⌘' },
];

const actGrid = sysCard.createDiv({ cls: 'komo-act-grid' });
ACTIONS.forEach(a => {
    const btn = actGrid.createDiv({ cls: 'komo-act-btn' });
    const ico = btn.createDiv({ cls: 'komo-act-icon' });
    try { setIcon(ico, a.icon); } catch(e) { ico.textContent = a.em; }
    btn.createDiv({ cls: 'komo-act-lbl', text: a.lbl });
    btn.addEventListener('click', () => {
        if (a.lbl === 'daily') {
            const today = new Date();
            const dateStr = today.toISOString().split('T')[0];
            const fileName = dateStr + '.md';
            const existingFile = app.vault.getAbstractFileByPath(fileName);
            if (existingFile) {
                app.workspace.openLinkText(fileName, '', false);
                new Notice('📅 Opened: ' + fileName);
            } else {
                app.vault.create(fileName, '# ' + today.toLocaleDateString() + '\n\n').then(() => {
                    app.workspace.openLinkText(fileName, '', false);
                    new Notice('📅 Created: ' + fileName);
                });
            }
            return;
        }
        try {
            if (!app.commands.commands[a.cmd]) {
                if (a.lbl === 'quick') new Notice('⚡ QuickAdd plugin not installed');
                else new Notice(`Command "${a.lbl}" not available`);
                return;
            }
            app.commands.executeCommandById(a.cmd);
        } catch(e) {
            new Notice(`Failed to run: ${a.lbl}`);
        }
    });
});

// ── WEATHER WIDGET (BIG) ─────────────────────
const weatherCard = colL.createDiv({ cls: 'komo-card komo-weather-card komo-big-widget' });

async function renderWeather() {
    const savedApiKey = localStorage.getItem('komo-weather-api-key') || '';
    const savedCity = localStorage.getItem('komo-weather-city') || 'barcelona';
    const savedUnits = localStorage.getItem('komo-weather-units') || 'metric';
    const savedLang = localStorage.getItem('komo-weather-lang') || 'en';

    weatherCard.innerHTML = '';
    weatherCard.createDiv({ cls: 'komo-label', text: 'weather' });

    // Settings button
    const settingsBtn = weatherCard.createDiv({ cls: 'komo-weather-settings' });
    try { setIcon(settingsBtn, 'settings'); } catch(e) { settingsBtn.textContent = '⚙'; }
    settingsBtn.addEventListener('click', showWeatherSettings);

    const weatherContent = weatherCard.createDiv({ cls: 'komo-weather-content' });

    if (!savedApiKey) {
        const setupMsg = weatherContent.createDiv({ cls: 'komo-weather-setup' });
        setupMsg.innerHTML = '<div class="komo-weather-big-icon">☁</div><div>configure weather</div>';
        setupMsg.addEventListener('click', showWeatherSettings);
        return;
    }

    try {
        const url = `https://api.openweathermap.org/data/2.5/weather?q=${savedCity}&units=${savedUnits}&lang=${savedLang}&appid=${savedApiKey}`;
        const response = await fetch(url);
        if (!response.ok) throw new Error('Weather error');
        const data = await response.json();

        const temp = Math.round(data.main.temp);
        const iconCode = data.weather[0].icon;
        const description = data.weather[0].description;
        const city = data.name;

        const mainRow = weatherContent.createDiv({ cls: 'komo-weather-main-big' });
        const iconEl = mainRow.createDiv({ cls: 'komo-weather-big-icon' });
        iconEl.innerHTML = getWeatherIcon(iconCode);

        const tempWrap = mainRow.createDiv({ cls: 'komo-weather-temp-wrap' });
        tempWrap.createDiv({ cls: 'komo-weather-temp-big', text: `${temp}°` });
        tempWrap.createDiv({ cls: 'komo-weather-city-big', text: city });

        weatherContent.createDiv({ cls: 'komo-weather-desc-big', text: description });

    } catch (e) {
        weatherContent.createDiv({ cls: 'komo-weather-error', text: '⚠ weather unavailable' });
    }
}

function getWeatherIcon(code) {
    const icons = {
        '01d': '☀', '01n': '☽',
        '02d': '⛅', '02n': '☁',
        '03d': '☁', '03n': '☁',
        '04d': '☁', '04n': '☁',
        '09d': '🌧', '09n': '🌧',
        '10d': '🌦', '10n': '🌧',
        '11d': '⚡', '11n': '⚡',
        '13d': '❄', '13n': '❄',
        '50d': '🌫', '50n': '🌫'
    };
    return icons[code] || '◌';
}

function showWeatherSettings() {
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal' });

    modal.createDiv({ cls: 'komo-modal-title', text: 'weather settings' });

    const form = modal.createDiv({ cls: 'komo-card-form' });

    const apiRow = form.createDiv({ cls: 'komo-form-row' });
    apiRow.createEl('label', { text: 'openweathermap api key' });
    const apiInput = apiRow.createEl('input', {
        cls: 'komo-modal-search',
        type: 'password',
        attr: { placeholder: 'your api key...', value: localStorage.getItem('komo-weather-api-key') || '' }
    });

    const cityRow = form.createDiv({ cls: 'komo-form-row' });
    cityRow.createEl('label', { text: 'city' });
    const cityInput = cityRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'city name...', value: localStorage.getItem('komo-weather-city') || 'barcelona' }
    });

    const unitRow = form.createDiv({ cls: 'komo-form-row' });
    unitRow.createEl('label', { text: 'units' });
    const unitSelect = unitRow.createEl('select', { cls: 'komo-modal-search' });
    ['metric', 'imperial', 'kelvin'].forEach(u => {
        const opt = unitSelect.createEl('option', { text: u, value: u });
        if (u === (localStorage.getItem('komo-weather-units') || 'metric')) opt.selected = true;
    });

    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: 'save' });

    cancelBtn.addEventListener('click', () => overlay.remove());
    saveBtn.addEventListener('click', () => {
        localStorage.setItem('komo-weather-api-key', apiInput.value.trim());
        localStorage.setItem('komo-weather-city', cityInput.value.trim() || 'barcelona');
        localStorage.setItem('komo-weather-units', unitSelect.value);
        renderWeather();
        overlay.remove();
    });

    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
}

renderWeather();

// ══════════════════════════════════════════════
//  CENTER — RELATED CARDS (BIG)
// ══════════════════════════════════════════════
const colC    = grid.createDiv({ cls: 'komo-col-center' });

// ══════════════════════════════════════════════
//  TASKS WIDGET
// ══════════════════════════════════════════════
const tasksCard = colC.createDiv({ cls: 'komo-card komo-tasks-card' });

// Load task pages from localStorage
let taskPages = JSON.parse(localStorage.getItem('komo-task-pages') || 'null') || [];
let currentTaskPage = parseInt(localStorage.getItem('komo-task-current-page') || '0');

// Load Completed Tasks plugin settings
let completedTasksSettings = null;

async function loadCompletedTasksSettings() {
    try {
        const settingsPath = '.obsidian/plugins/completed-tasks/data.json';
        const content = await app.vault.adapter.read(settingsPath);
        completedTasksSettings = JSON.parse(content);
        console.log('Loaded completed-tasks settings:', completedTasksSettings);
    } catch (e) {
        console.log('Could not load completed-tasks settings:', e);
        completedTasksSettings = null;
    }
}

await loadCompletedTasksSettings();

// Header with label, pagination, and settings
const tasksHeader = tasksCard.createDiv({ cls: 'komo-tasks-header' });
tasksHeader.createDiv({ cls: 'komo-label', text: 'tasks' });

const paginationControls = tasksHeader.createDiv({ cls: 'komo-tasks-pagination' });

// Settings button (gear icon)
const tasksSettingsBtn = paginationControls.createDiv({ cls: 'komo-tasks-settings-btn' });
try { setIcon(tasksSettingsBtn, 'settings'); } catch(e) { tasksSettingsBtn.textContent = '⚙'; }

// Tasks container
const tasksContainer = tasksCard.createDiv({ cls: 'komo-tasks-container' });

// Parse task tree from lines
function parseTaskTree(lines) {
    const roots = [];
    const stack = [];
    
    lines.forEach((raw, idx) => {
        const match = raw.match(/^(\t*)\s*- \[([ xX])\]\s*(.*)/);
        if (!match) return;
        
        const indent = match[1].length;
        const done = match[2].toLowerCase() === 'x';
        let text = match[3].trim();
        
        const node = {
            lineIndex: idx,
            indent,
            done,
            rawText: text,
            children: []
        };
        
        while (stack.length > 0 && stack[stack.length - 1].indent >= indent) {
            stack.pop();
        }
        
        if (stack.length === 0) {
            roots.push(node);
        } else {
            stack[stack.length - 1].node.children.push(node);
        }
        
        stack.push({ indent, node });
    });
    
    return roots;
}

// Checkbox toggle with Tasks plugin support
let isModifying = false;

async function toggleTaskLine(filePath, lineIndex) {
    if (isModifying) return;
    isModifying = true;
    
    try {
        const file = app.vault.getAbstractFileByPath(filePath);
        if (!file) return;
        
        let content = await app.vault.read(file);
        const lines = content.split('\n');
        if (lineIndex >= lines.length) return;
        
        const line = lines[lineIndex];
        const isTopLevel = /^- \[/.test(line); // Only sort if top-level task
        
        if (line.includes('- [ ]')) {
            const hasTasksMetadata = /[📅⏫🔼🔽⏬✅❌🔁📆🔺]/.test(line) || /\d{4}-\d{2}-\d{2}/.test(line);
            
            if (hasTasksMetadata) {
                const today = new Date().toISOString().split('T')[0];
                lines[lineIndex] = line.replace('- [ ]', '- [x]') + ` ✅ ${today}`;
            } else {
                lines[lineIndex] = line.replace('- [ ]', '- [x]');
            }
            
        } else if (line.includes('- [x]') || line.includes('- [X]')) {
            let newLine = line.replace(/- \[[xX]\]/, '- [ ]');
            newLine = newLine.replace(/\s*✅\s*\d{4}-\d{2}-\d{2}/, '');
            lines[lineIndex] = newLine;
        }
        
        await app.vault.modify(file, lines.join('\n'));
        
        // Only sort if this was a top-level task toggle
        if (isTopLevel) {
            await sortCompletedTasks(filePath);
        }
    } finally {
        setTimeout(() => { isModifying = false; }, 200);
    }
}

// Completed Tasks sorting integration - sorts directly without navigating
async function sortCompletedTasks(filePath) {
    if (!completedTasksSettings) {
        console.log('No completed tasks settings loaded');
        return;
    }
    
    const file = app.vault.getAbstractFileByPath(filePath);
    if (!file) {
        console.log('File not found:', filePath);
        return;
    }
    
    // Small delay to ensure the toggle write has completed
    await new Promise(resolve => setTimeout(resolve, 50));
    
    let content = await app.vault.read(file);
    const lines = content.split('\n');
    
    // Check for ignore substring in file
    const ignoreSubstrings = completedTasksSettings.ignoreSubstrings || ['#donotsort'];
    if (ignoreSubstrings.some(s => content.includes(s))) {
        console.log('File has ignore substring, skipping sort');
        return;
    }
    
    const statuses = completedTasksSettings.statuses || ['- [ ]', '- [/]', '- [x]', '- [-]', '- [>]', '- [<]'];
    const sortedStatuses = completedTasksSettings.sortedStatuses || ['- [ ]', '- [/]', '- [x]'];
    const sortedSubstrings = completedTasksSettings.sortedSubstrings || ['⏬', '🔽', '🔼', '⏫', '🔺'];
    
    function isTaskLine(line) {
        const trimmed = line.trimStart();
        return statuses.some(s => trimmed.startsWith(s));
    }
    
    function isTopLevelTask(line) {
        // Top-level task: starts with "- [" (no leading tabs/spaces for indentation)
        return /^- \[[ xX\/\-><]\]/.test(line);
    }
    
    function isSubtask(line) {
        // Subtask: indented task (starts with tab or spaces before "- [")
        return isTaskLine(line) && !isTopLevelTask(line);
    }
    
    function getStatusSortVal(line) {
        const trimmed = line.trimStart();
        for (let i = 0; i < sortedStatuses.length; i++) {
            if (trimmed.startsWith(sortedStatuses[i])) return i + 1;
        }
        return 0; // Not a sorted status
    }
    
    function getSubSortVal(line) {
        const trimmed = line.trimStart();
        // sortedSubstrings: ['⏬', '🔽', '🔼', '⏫', '🔺']
        // Index 4 (🔺) = highest priority = should sort FIRST
        // Use negative index directly: 🔺 = -4, ⏬ = 0
        for (let i = 0; i < sortedSubstrings.length; i++) {
            if (trimmed.includes(sortedSubstrings[i])) {
                return -i; // Higher index = more negative = sorts first
            }
        }
        return 1; // No priority = positive = sorts after all priorities
    }
    
    // Group tasks: each top-level task includes its subtasks
    const result = [];
    let i = 0;
    
    while (i < lines.length) {
        if (isTopLevelTask(lines[i])) {
            // Found a top-level task - collect it with all its subtasks
            const taskGroup = {
                lines: [lines[i]],
                statusVal: getStatusSortVal(lines[i]),
                subVal: getSubSortVal(lines[i])
            };
            i++;
            
            // Collect all following subtasks (indented tasks)
            while (i < lines.length && isSubtask(lines[i])) {
                taskGroup.lines.push(lines[i]);
                i++;
            }
            
            result.push({ type: 'task', data: taskGroup });
        } else if (isTaskLine(lines[i])) {
            // Orphan subtask (shouldn't happen, but handle it)
            result.push({ type: 'line', data: lines[i] });
            i++;
        } else {
            // Non-task line - marks block boundary
            result.push({ type: 'line', data: lines[i] });
            i++;
        }
    }
    
    // Find consecutive task groups and sort them
    const finalLines = [];
    let taskBuffer = [];
    
    for (const item of result) {
        if (item.type === 'task') {
            taskBuffer.push(item.data);
        } else {
            // Non-task line - flush and sort the task buffer
            if (taskBuffer.length > 0) {
                taskBuffer.sort((a, b) => {
                    if (a.statusVal !== b.statusVal) {
                        return a.statusVal - b.statusVal;
                    }
                    return a.subVal - b.subVal;
                });
                taskBuffer.forEach(tg => tg.lines.forEach(l => finalLines.push(l)));
                taskBuffer = [];
            }
            finalLines.push(item.data);
        }
    }
    
    // Flush remaining tasks
    if (taskBuffer.length > 0) {
        taskBuffer.sort((a, b) => {
            if (a.statusVal !== b.statusVal) {
                return a.statusVal - b.statusVal;
            }
            return a.subVal - b.subVal;
        });
        taskBuffer.forEach(tg => tg.lines.forEach(l => finalLines.push(l)));
    }
    
    const newContent = finalLines.join('\n');
    if (newContent !== content) {
        console.log('Sorting tasks in', filePath);
        await app.vault.modify(file, newContent);
    }
}

// Edit task inline
async function editTaskLine(filePath, lineIndex, currentText, refreshFn) {
    const file = app.vault.getAbstractFileByPath(filePath);
    if (!file) return;
    
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal komo-edit-task-modal' });
    
    modal.createDiv({ cls: 'komo-modal-title', text: 'edit task' });
    
    const form = modal.createDiv({ cls: 'komo-card-form' });
    const inputRow = form.createDiv({ cls: 'komo-form-row' });
    
    const input = inputRow.createEl('input', {
        cls: 'komo-modal-input',
        attr: { 
            type: 'text', 
            value: currentText,
            placeholder: 'Task description...'
        }
    });
    
    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: 'save' });
    
    cancelBtn.addEventListener('click', () => overlay.remove());
    
    saveBtn.addEventListener('click', async () => {
        const newText = input.value.trim();
        if (newText && newText !== currentText) {
            let content = await app.vault.read(file);
            const lines = content.split('\n');
            
            const line = lines[lineIndex];
            const match = line.match(/^(\t*\s*- \[[ xX]\]\s*)(.*)/);
            if (match) {
                lines[lineIndex] = match[1] + newText;
                await app.vault.modify(file, lines.join('\n'));
            }
        }
        overlay.remove();
        if (refreshFn) setTimeout(refreshFn, 100);
    });
    
    input.addEventListener('keydown', (e) => {
        if (e.key === 'Enter') {
            e.preventDefault();
            saveBtn.click();
        }
    });
    
    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
    setTimeout(() => input.focus(), 50);
}

// Delete task
async function deleteTaskLine(filePath, lineIndex, refreshFn) {
    const file = app.vault.getAbstractFileByPath(filePath);
    if (!file) return;
    
    let content = await app.vault.read(file);
    const lines = content.split('\n');
    lines.splice(lineIndex, 1);
    
    await app.vault.modify(file, lines.join('\n'));
    if (refreshFn) setTimeout(refreshFn, 100);
}

// Context menu
function showTaskContextMenu(event, filePath, lineIndex, currentText, refreshFn) {
    const menu = new obsidian.Menu();
    
    menu.addItem((item) => {
        item.setTitle('Edit')
            .setIcon('pencil')
            .onClick(() => {
                editTaskLine(filePath, lineIndex, currentText, refreshFn);
            });
    });
    
    menu.addItem((item) => {
        item.setTitle('Delete')
            .setIcon('trash')
            .onClick(async () => {
                await deleteTaskLine(filePath, lineIndex, refreshFn);
            });
    });
    
    menu.showAtMouseEvent(event);
}

// Render task node
function renderTaskNode(container, node, filePath, refreshFn) {
    if (node.children.length > 0) {
        const details = container.createEl('details', { cls: 'komo-task-details' });
        
        // Use unique key for collapse state persistence
        const collapseKey = `komo-collapse-${filePath}-${node.lineIndex}`;
        const savedState = localStorage.getItem(collapseKey);
        details.open = savedState === 'open'; // Default closed unless saved as open
        
        const summary = details.createEl('summary', { cls: 'komo-task-item komo-task-parent' });
        
        // Collapse indicator
        const collapseIndicator = summary.createEl('span', { cls: 'komo-task-collapse-indicator' });
        collapseIndicator.textContent = details.open ? '▾' : '▸';
        
        details.addEventListener('toggle', () => {
            collapseIndicator.textContent = details.open ? '▾' : '▸';
            // Save collapse state
            localStorage.setItem(collapseKey, details.open ? 'open' : 'closed');
        });
        
        const checkbox = summary.createEl('input', { 
            type: 'checkbox', 
            cls: 'komo-task-checkbox' 
        });
        checkbox.checked = node.done;
        checkbox.addEventListener('click', async (e) => {
            e.stopPropagation();
            // Toggle visually immediately
            node.done = !node.done;
            checkbox.checked = node.done;
            label.classList.toggle('komo-task-done', node.done);
            // Update file in background (no page refresh)
            await toggleTaskLine(filePath, node.lineIndex);
        });
        
        const label = summary.createEl('span', {
            cls: `komo-task-text ${node.done ? 'komo-task-done' : ''}`
        });
        label.textContent = node.rawText;
        
        label.addEventListener('click', (e) => {
            e.stopPropagation();
            e.preventDefault();
            editTaskLine(filePath, node.lineIndex, node.rawText, refreshFn);
        });
        
        label.addEventListener('contextmenu', (e) => {
            e.preventDefault();
            showTaskContextMenu(e, filePath, node.lineIndex, node.rawText, refreshFn);
        });
        
        const childContainer = details.createDiv({ cls: 'komo-task-children' });
        node.children.forEach(child => renderTaskNode(childContainer, child, filePath, refreshFn));
        
    } else {
        const item = container.createDiv({ cls: 'komo-task-item' });
        
        const checkbox = item.createEl('input', { 
            type: 'checkbox', 
            cls: 'komo-task-checkbox' 
        });
        checkbox.checked = node.done;
        checkbox.addEventListener('click', async (e) => {
            e.stopPropagation();
            // Toggle visually immediately
            node.done = !node.done;
            checkbox.checked = node.done;
            label.classList.toggle('komo-task-done', node.done);
            // Update file in background (no page refresh)
            await toggleTaskLine(filePath, node.lineIndex);
        });
        
        const label = item.createEl('span', {
            cls: `komo-task-text ${node.done ? 'komo-task-done' : ''}`
        });
        label.textContent = node.rawText;
        
        label.addEventListener('click', (e) => {
            e.stopPropagation();
            editTaskLine(filePath, node.lineIndex, node.rawText, refreshFn);
        });
        
        label.addEventListener('contextmenu', (e) => {
            e.preventDefault();
            showTaskContextMenu(e, filePath, node.lineIndex, node.rawText, refreshFn);
        });
    }
}

// Update pagination dots
function updatePaginationDots() {
    const existingDots = paginationControls.querySelector('.komo-pagination-dots');
    if (existingDots) existingDots.remove();
    
    if (taskPages.length > 1) {
        const dotsContainer = document.createElement('div');
        dotsContainer.className = 'komo-pagination-dots';
        taskPages.forEach((_, idx) => {
            const dot = dotsContainer.createDiv({ 
                cls: `komo-pagination-dot ${idx === currentTaskPage ? 'active' : ''}` 
            });
            dot.addEventListener('click', () => {
                currentTaskPage = idx;
                localStorage.setItem('komo-task-current-page', currentTaskPage);
                renderTaskPage();
            });
        });
        // Insert dots BEFORE the settings button (first child)
        paginationControls.insertBefore(dotsContainer, paginationControls.firstChild);
    }
}

// Settings modal
function showTasksSettingsModal() {
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal komo-tasks-settings-modal' });
    
    modal.createDiv({ cls: 'komo-modal-title', text: 'task pages settings' });
    
    const content = modal.createDiv({ cls: 'komo-tasks-settings-content' });
    content.createDiv({ 
        cls: 'komo-tasks-settings-info', 
        text: 'Configure up to 4 pages, each with up to 4 task files.' 
    });
    
    const pagesContainer = content.createDiv({ cls: 'komo-pages-container' });
    
    function renderPageSettings() {
        pagesContainer.innerHTML = '';
        
        taskPages.forEach((page, pageIdx) => {
            const pageBlock = pagesContainer.createDiv({ cls: 'komo-page-block' });
            const pageHeader = pageBlock.createDiv({ cls: 'komo-page-header' });
            pageHeader.createDiv({ text: `Page ${pageIdx + 1}`, cls: 'komo-page-title' });
            
            if (taskPages.length > 1) {
                const removePageBtn = pageHeader.createDiv({ cls: 'komo-remove-page-btn', text: '✕' });
                removePageBtn.addEventListener('click', () => {
                    taskPages.splice(pageIdx, 1);
                    if (currentTaskPage >= taskPages.length) currentTaskPage = taskPages.length - 1;
                    renderPageSettings();
                });
            }
            
            const filesContainer = pageBlock.createDiv({ cls: 'komo-files-container' });
            
            page.forEach((filePath, fileIdx) => {
                const fileRow = filesContainer.createDiv({ cls: 'komo-file-row' });
                
                const allFiles = app.vault.getMarkdownFiles();
                const filePaths = allFiles.map(f => f.path).sort();
                
                const fileInput = fileRow.createEl('input', {
                    cls: 'komo-modal-input',
                    attr: { 
                        type: 'text', 
                        value: filePath, 
                        placeholder: 'Type or select file...',
                        list: `file-list-${pageIdx}-${fileIdx}`
                    }
                });
                
                const datalist = fileRow.createEl('datalist', {
                    attr: { id: `file-list-${pageIdx}-${fileIdx}` }
                });
                
                filePaths.forEach(path => {
                    datalist.createEl('option', { value: path });
                });
                
                fileInput.addEventListener('change', () => {
                    taskPages[pageIdx][fileIdx] = fileInput.value.trim();
                });
                
                const removeFileBtn = fileRow.createDiv({ cls: 'komo-remove-file-btn', text: '✕' });
                removeFileBtn.addEventListener('click', () => {
                    taskPages[pageIdx].splice(fileIdx, 1);
                    renderPageSettings();
                });
            });
            
            if (page.length < 4) {
                const addFileBtn = pageBlock.createDiv({ cls: 'komo-add-file-btn', text: '+ add file' });
                addFileBtn.addEventListener('click', () => {
                    taskPages[pageIdx].push('');
                    renderPageSettings();
                });
            }
        });
        
        if (taskPages.length < 4) {
            const addPageBtn = pagesContainer.createDiv({ cls: 'komo-add-page-btn', text: '+ add page' });
            addPageBtn.addEventListener('click', () => {
                taskPages.push([]);
                renderPageSettings();
            });
        }
    }
    
    renderPageSettings();
    
    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: 'save' });
    
    cancelBtn.addEventListener('click', () => {
        taskPages = JSON.parse(localStorage.getItem('komo-task-pages') || 'null') || [];
        overlay.remove();
    });
    
    saveBtn.addEventListener('click', () => {
        taskPages = taskPages
            .map(page => page.filter(path => path.trim()))
            .filter(page => page.length > 0);
        
        if (currentTaskPage >= taskPages.length && taskPages.length > 0) {
            currentTaskPage = 0;
        }
        
        localStorage.setItem('komo-task-pages', JSON.stringify(taskPages));
        localStorage.setItem('komo-task-current-page', currentTaskPage);
        overlay.remove();
        renderTaskPage();
    });
    
    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
}

// Render current task page
async function renderTaskPage() {
    tasksContainer.innerHTML = '';
    
    if (taskPages.length === 0) {
        const emptyState = tasksContainer.createDiv({ cls: 'komo-tasks-empty-state' });
        emptyState.innerHTML = '<div>no task pages configured</div>' +
            '<div style="font-size: 0.65rem; color: var(--komo-muted); margin-top: 8px;">' +
            'click ⚙ to add task files</div>';
        updatePaginationDots();
        return;
    }
    
    if (currentTaskPage >= taskPages.length) {
        currentTaskPage = 0;
        localStorage.setItem('komo-task-current-page', currentTaskPage);
    }
    
    const currentPage = taskPages[currentTaskPage] || [];
    
    for (const path of currentPage) {
        const col = tasksContainer.createDiv({ cls: 'komo-task-col' });
        
        const colHeader = col.createDiv({ cls: 'komo-task-col-header' });
        const colTitle = colHeader.createDiv({ cls: 'komo-task-col-title' });
        const fileName = path.split('/').pop().replace('.md', '');
        colTitle.textContent = fileName;
        
        const addTaskBtn = colHeader.createDiv({ cls: 'komo-add-task-btn' });
        addTaskBtn.textContent = '+';
        
        addTaskBtn.addEventListener('click', async () => {
            const file = app.vault.getAbstractFileByPath(path);
            if (!file) {
                new Notice('File not found: ' + path);
                return;
            }
            
            const leaf = app.workspace.getLeaf(false);
            await leaf.openFile(file);
            
            // Wait for file to fully load and enter editing mode
            setTimeout(async () => {
                const editor = app.workspace.activeLeaf?.view?.editor;
                if (editor) {
                    // Move cursor to end of file
                    const lastLine = editor.lastLine();
                    editor.setCursor({ line: lastLine, ch: editor.getLine(lastLine).length });
                    
                    // Press Enter to create new line first
                    const lineContent = editor.getLine(lastLine);
                    editor.replaceRange('\n', { line: lastLine, ch: lineContent.length });
                    editor.setCursor({ line: lastLine + 1, ch: 0 });
                    
                    // Small delay after creating newline before command palette
                    setTimeout(() => {
                        // Open command palette
                        app.commands.executeCommandById('command-palette:open');
                        
                        setTimeout(() => {
                            const paletteInput = document.querySelector('.prompt-input');
                            if (paletteInput) {
                                paletteInput.value = 'Tasks: Create';
                                paletteInput.dispatchEvent(new Event('input', { bubbles: true }));
                            }
                            
                            setTimeout(() => {
                                // Press Enter to execute command
                                const enterEvent = new KeyboardEvent('keydown', {
                                    key: 'Enter',
                                    code: 'Enter',
                                    keyCode: 13,
                                    which: 13,
                                    bubbles: true
                                });
                                document.activeElement?.dispatchEvent(enterEvent);
                            }, 60);
                        }, 80);
                    }, 100);
                }
            }, 400);
        });
        
        const colContent = col.createDiv({ cls: 'komo-task-col-content' });
        
        const file = app.vault.getAbstractFileByPath(path);
        if (!file) {
            colContent.createDiv({ 
                text: 'File not found', 
                cls: 'komo-tasks-empty-state',
                attr: { style: 'min-height: 80px; font-size: 0.7rem;' }
            });
            continue;
        }
        
        const content = await app.vault.read(file);
        const lines = content.split('\n');
        const taskTree = parseTaskTree(lines);
        
        if (taskTree.length === 0) {
            colContent.createDiv({ 
                text: 'No tasks yet', 
                cls: 'komo-tasks-empty-state',
                attr: { style: 'min-height: 80px; font-size: 0.7rem;' }
            });
        } else {
            taskTree.forEach(node => {
                renderTaskNode(colContent, node, path, renderTaskPage);
            });
        }
    }
    
    updatePaginationDots();
}

// Event handlers
tasksSettingsBtn.addEventListener('click', showTasksSettingsModal);

// Initial render
renderTaskPage();

// ══════════════════════════════════════════════
//  CENTER — RELATED CARDS (BIG)
// ══════════════════════════════════════════════
const cardsCard = colC.createDiv({ cls: 'komo-card komo-cards-card' });

// Header with add button
const cardsHdr = cardsCard.createDiv({ cls: 'komo-cards-hdr' });
cardsHdr.createDiv({ cls: 'komo-label', text: 'collection' });

const addCardBtn = cardsHdr.createDiv({ cls: 'komo-add-card-btn', text: '+ add' });
addCardBtn.addEventListener('click', showAddCardModal);

// Cards container
const cardsContainer = cardsCard.createDiv({ cls: 'komo-cards-container komo-cards-big' });

// Load cards from localStorage
let cards = JSON.parse(localStorage.getItem('komo-cards') || '[]');

// Get all available files for dropdown
function getAllFiles() {
    return app.vault.getMarkdownFiles()
        .map(f => ({ name: f.name, path: f.path }))
        .sort((a, b) => a.name.localeCompare(b.name));
}

// Build a single card DOM element
function buildCardEl(card, idx) {
    const cardEl = document.createElement('div');
    cardEl.className = 'komo-related-card komo-card-big';
    cardEl.style.borderColor = card.color || 'var(--komo-border)';
    cardEl.dataset.cardIdx = idx;

    const imgWrap = document.createElement('div');
    imgWrap.className = 'komo-card-img-wrap-big';
    if (card.image) {
        const img = document.createElement('img');
        img.className = 'komo-card-img-big';
        if (card.image.startsWith('http://') || card.image.startsWith('https://')) {
            img.src = card.image;
        } else {
            try { img.src = app.vault.adapter.getResourcePath(card.image); }
            catch(e) { img.src = card.image; }
        }
        img.onerror = () => { imgWrap.innerHTML = ''; imgWrap.textContent = card.emoji || '◻'; };
        imgWrap.appendChild(img);
    } else {
        imgWrap.textContent = card.emoji || '◻';
    }
    cardEl.appendChild(imgWrap);

    const content = document.createElement('div');
    content.className = 'komo-card-content-big';
    const titleEl2 = document.createElement('div');
    titleEl2.className = 'komo-card-title-big';
    titleEl2.textContent = card.title;
    content.appendChild(titleEl2);
    if (card.subtitle) {
        const subEl = document.createElement('div');
        subEl.className = 'komo-card-subtitle-big';
        subEl.textContent = card.subtitle;
        content.appendChild(subEl);
    }
    cardEl.appendChild(content);

    const actions = document.createElement('div');
    actions.className = 'komo-card-actions';
    const editBtn = document.createElement('div');
    editBtn.className = 'komo-card-action';
    editBtn.textContent = '✎';
    const delBtn2 = document.createElement('div');
    delBtn2.className = 'komo-card-action komo-card-del';
    delBtn2.textContent = '✕';
    actions.appendChild(editBtn);
    actions.appendChild(delBtn2);
    cardEl.appendChild(actions);

    editBtn.addEventListener('click', (e) => { e.stopPropagation(); editCard(idx); });
    delBtn2.addEventListener('click', (e) => { e.stopPropagation(); deleteCard(idx); });

    if (card.link) {
        cardEl.addEventListener('click', () => app.workspace.openLinkText(card.link, '', false));
        cardEl.style.cursor = 'pointer';
    }
    return cardEl;
}

// Full render — only called on first load
function renderCards() {
    cardsContainer.innerHTML = '';
    if (cards.length === 0) {
        const empty = cardsContainer.createDiv({ cls: 'komo-cards-empty' });
        empty.createEl('span', { cls: 'komo-empty-icon', text: '◻' });
        empty.createEl('span', { text: 'no cards yet — click +add to create one' });
        return;
    }
    cards.forEach((card, idx) => cardsContainer.appendChild(buildCardEl(card, idx)));
}

// Surgical update — replaces only ONE card node, never touches others
function patchCard(idx) {
    cards = JSON.parse(localStorage.getItem('komo-cards') || '[]');
    if (!cardsContainer.isConnected) { renderCards(); return; }

    const existing = cardsContainer.querySelector(`[data-card-idx="${idx}"]`);
    if (existing) {
        const newEl = buildCardEl(cards[idx], idx);
        cardsContainer.replaceChild(newEl, existing);
    } else {
        // Card is new — append it and remove empty placeholder if present
        const empty = cardsContainer.querySelector('.komo-cards-empty');
        if (empty) empty.remove();
        cardsContainer.appendChild(buildCardEl(cards[idx], idx));
    }
    // Re-index all cards so their idx stays correct after any reorder
    Array.from(cardsContainer.children).forEach((el, i) => { el.dataset.cardIdx = i; });
}

function showAddCardModal() {
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal komo-card-modal' });

    modal.createDiv({ cls: 'komo-modal-title', text: 'create card' });

    const form = modal.createDiv({ cls: 'komo-card-form' });

    const titleRow = form.createDiv({ cls: 'komo-form-row' });
    titleRow.createEl('label', { text: 'title' });
    const titleInput = titleRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'card name...', type: 'text' }
    });

    const subRow = form.createDiv({ cls: 'komo-form-row' });
    subRow.createEl('label', { text: 'subtitle (optional)' });
    const subInput = subRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'description...', type: 'text' }
    });

    const emojiRow = form.createDiv({ cls: 'komo-form-row' });
    emojiRow.createEl('label', { text: 'emoji' });
    const emojiInput = emojiRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: '🌸', type: 'text', maxlength: '2' }
    });

    const imgRow = form.createDiv({ cls: 'komo-form-row' });
    imgRow.createEl('label', { text: 'image path (optional)' });
    const imgInput = imgRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'path/to/image.png or https://...', type: 'text' }
    });

    const linkRow = form.createDiv({ cls: 'komo-form-row' });
    linkRow.createEl('label', { text: 'link to note' });
    const linkSelect = linkRow.createEl('select', { cls: 'komo-modal-search' });
    linkSelect.createEl('option', { text: '-- no link --', value: '' });
    getAllFiles().forEach(f => linkSelect.createEl('option', { text: f.name, value: f.path }));

    const colorRow = form.createDiv({ cls: 'komo-form-row' });
    colorRow.createEl('label', { text: 'accent color' });
    const colorGrid = colorRow.createDiv({ cls: 'komo-color-grid' });
    const colors = ['#f5c2e7', '#cba6f7', '#89dceb', '#f9e2af', '#f38ba8', '#a6e3a1', '#fab387'];
    let selectedColor = colors[0];
    colors.forEach(c => {
        const colorDot = colorGrid.createDiv({ cls: 'komo-color-dot' });
        colorDot.style.background = c;
        if (c === selectedColor) colorDot.classList.add('active');
        colorDot.addEventListener('click', () => {
            colorGrid.querySelectorAll('.komo-color-dot').forEach(d => d.classList.remove('active'));
            colorDot.classList.add('active');
            selectedColor = c;
        });
    });

    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: 'create' });

    cancelBtn.addEventListener('click', () => overlay.remove());
    saveBtn.addEventListener('click', () => {
        const newCard = {
            id: Date.now(),
            title: titleInput.value.trim() || 'untitled',
            subtitle: subInput.value.trim(),
            emoji: emojiInput.value.trim(),
            image: imgInput.value.trim(),
            link: linkSelect.value,
            color: selectedColor
        };
        cards.push(newCard);
        localStorage.setItem('komo-cards', JSON.stringify(cards));
        overlay.remove();
        patchCard(cards.length - 1);
    });

    titleInput.focus();
    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
}

function editCard(idx) {
    const card = cards[idx];
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal komo-card-modal' });

    modal.createDiv({ cls: 'komo-modal-title', text: 'edit card' });

    const form = modal.createDiv({ cls: 'komo-card-form' });

    const titleRow = form.createDiv({ cls: 'komo-form-row' });
    titleRow.createEl('label', { text: 'title' });
    const titleInput = titleRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'card name...', type: 'text', value: card.title }
    });

    const subRow = form.createDiv({ cls: 'komo-form-row' });
    subRow.createEl('label', { text: 'subtitle (optional)' });
    const subInput = subRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'description...', type: 'text', value: card.subtitle || '' }
    });

    const emojiRow = form.createDiv({ cls: 'komo-form-row' });
    emojiRow.createEl('label', { text: 'emoji' });
    const emojiInput = emojiRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: '🌸', type: 'text', maxlength: '2', value: card.emoji || '' }
    });

    const imgRow = form.createDiv({ cls: 'komo-form-row' });
    imgRow.createEl('label', { text: 'image path (optional)' });
    const imgInput = imgRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'path/to/image.png or https://...', type: 'text', value: card.image || '' }
    });

    // File selector dropdown
    const linkRow = form.createDiv({ cls: 'komo-form-row' });
    linkRow.createEl('label', { text: 'link to note' });

    const linkSelect = linkRow.createEl('select', { cls: 'komo-modal-search' });
    linkSelect.createEl('option', { text: '-- no link --', value: '' });

    const allFiles = getAllFiles();
    allFiles.forEach(f => {
        const opt = linkSelect.createEl('option', { text: f.name, value: f.path });
        if (f.path === card.link) opt.selected = true;
    });

    const colorRow = form.createDiv({ cls: 'komo-form-row' });
    colorRow.createEl('label', { text: 'accent color' });
    const colorGrid = colorRow.createDiv({ cls: 'komo-color-grid' });
    const colors = ['#f5c2e7', '#cba6f7', '#89dceb', '#f9e2af', '#f38ba8', '#a6e3a1', '#fab387'];
    let selectedColor = card.color || colors[0];

    colors.forEach(c => {
        const colorDot = colorGrid.createDiv({ cls: 'komo-color-dot' });
        colorDot.style.background = c;
        if (c === selectedColor) colorDot.classList.add('active');
        colorDot.addEventListener('click', () => {
            colorGrid.querySelectorAll('.komo-color-dot').forEach(d => d.classList.remove('active'));
            colorDot.classList.add('active');
            selectedColor = c;
        });
    });

    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: 'save' });
    const delBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-delete', text: 'delete' });

    cancelBtn.addEventListener('click', () => overlay.remove());
    saveBtn.addEventListener('click', () => {
        cards[idx] = {
            ...card,
            title: titleInput.value.trim() || 'untitled',
            subtitle: subInput.value.trim(),
            emoji: emojiInput.value.trim(),
            image: imgInput.value.trim(),
            link: linkSelect.value,
            color: selectedColor
        };
        localStorage.setItem('komo-cards', JSON.stringify(cards));
        overlay.remove();
        patchCard(idx);
    });
    delBtn.addEventListener('click', () => {
        // Remove the DOM node directly — no full re-render needed
        const el = cardsContainer.querySelector(`[data-card-idx="${idx}"]`);
        if (el) el.remove();
        cards.splice(idx, 1);
        localStorage.setItem('komo-cards', JSON.stringify(cards));
        overlay.remove();
        // Re-index remaining cards
        Array.from(cardsContainer.children).forEach((e, i) => { e.dataset.cardIdx = i; });
        if (cards.length === 0) {
            const empty = cardsContainer.createDiv({ cls: 'komo-cards-empty' });
            empty.createEl('span', { cls: 'komo-empty-icon', text: '◻' });
            empty.createEl('span', { text: 'no cards yet — click +add to create one' });
        }
    });

    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
}

function showConfirmModal(message, onConfirm, onCancel) {
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal komo-confirm-modal' });

    const content = modal.createDiv({ cls: 'komo-confirm-content' });
    content.createDiv({ cls: 'komo-confirm-icon', text: '⚠' });
    content.createDiv({ cls: 'komo-confirm-title', text: 'confirm deletion' });
    content.createDiv({ cls: 'komo-confirm-message', text: message });

    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const confirmBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-delete', text: 'delete' });

    cancelBtn.addEventListener('click', () => {
        overlay.remove();
        if (onCancel) onCancel();
    });

    confirmBtn.addEventListener('click', () => {
        overlay.remove();
        if (onConfirm) onConfirm();
    });

    overlay.addEventListener('click', e => {
        if (e.target === overlay) {
            overlay.remove();
            if (onCancel) onCancel();
        }
    });
}

function deleteCard(idx) {
    const card = cards[idx];
    showConfirmModal(`delete "${card.title}"?`, () => {
        const el = cardsContainer.querySelector(`[data-card-idx="${idx}"]`);
        if (el) el.remove();
        cards.splice(idx, 1);
        localStorage.setItem('komo-cards', JSON.stringify(cards));
        Array.from(cardsContainer.children).forEach((e, i) => { e.dataset.cardIdx = i; });
        if (cards.length === 0) {
            const empty = cardsContainer.createDiv({ cls: 'komo-cards-empty' });
            empty.createEl('span', { cls: 'komo-empty-icon', text: '◻' });
            empty.createEl('span', { text: 'no cards yet — click +add to create one' });
        }
    });
}

renderCards();

// ══════════════════════════════════════════════
//  RIGHT COLUMN — CALENDAR ONLY
// ══════════════════════════════════════════════
const colR = grid.createDiv({ cls: 'komo-col-right' });

// ── MINI CALENDAR (daily notes only) ─────
const calCard = colR.createDiv({ cls: 'komo-card komo-cal-card' });
calCard.createDiv({ cls: 'komo-label', text: 'calendar' });

let calY = new Date().getFullYear();
let calM = new Date().getMonth();
const calContainer = calCard.createDiv({ cls: 'komo-cal-container' });

// Get daily notes for date (only YYYY-MM-DD format)
function getDailyNotesForDate(year, month, day) {
    const dateStr = `${year}-${String(month+1).padStart(2,'0')}-${String(day).padStart(2,'0')}`;
    const notes = [];

    const allFiles = app.vault.getMarkdownFiles();
    allFiles.forEach(f => {
        // Only match exact date format YYYY-MM-DD
        if (f.basename === dateStr) {
            notes.push({ name: f.basename, path: f.path, type: 'daily' });
        }
    });

    return notes;
}

function renderCal() {
    calContainer.innerHTML = '';

    // Navigation
    const nav  = calContainer.createDiv({ cls: 'komo-cal-nav' });
    const prev = nav.createDiv({ cls: 'komo-cal-nav-btn', text: '‹' });
    nav.createDiv({
        cls: 'komo-cal-month-label',
        text: new Date(calY, calM).toLocaleDateString('en-US', { year: 'numeric', month: 'long' })
    });
    const next = nav.createDiv({ cls: 'komo-cal-nav-btn', text: '›' });

    prev.addEventListener('click', () => { if(calM===0){calM=11;calY--;}else calM--; renderCal(); });
    next.addEventListener('click', () => { if(calM===11){calM=0;calY++;}else calM++; renderCal(); });

    // Day headers (English, Sun–Sat)
    const hdrG = calContainer.createDiv({ cls: 'komo-cal-grid' });
    ['Su','Mo','Tu','We','Th','Fr','Sa'].forEach(d => hdrG.createDiv({ cls: 'komo-cal-dh', text: d }));

    // Day cells
    const dayG     = calContainer.createDiv({ cls: 'komo-cal-grid' });
    const firstDay = new Date(calY, calM, 1).getDay();   // 0=Sun
    const dim      = new Date(calY, calM + 1, 0).getDate();
    const now      = new Date();

    for (let i = 0; i < firstDay; i++) {
        dayG.createDiv({ cls: 'komo-cal-cell empty' });
    }
    for (let d = 1; d <= dim; d++) {
        const isToday = d === now.getDate() && calM === now.getMonth() && calY === now.getFullYear();
        const cell    = dayG.createDiv({ cls: `komo-cal-cell ${isToday ? 'today' : ''}` });

        // Day number
        const dayNum = cell.createDiv({ cls: 'komo-cal-day-num', text: String(d) });

        // Check for daily notes on this day
        const notes = getDailyNotesForDate(calY, calM, d);

        // Add pink dot if there are daily notes
        if (notes.length > 0) {
            const dotContainer = cell.createDiv({ cls: 'komo-cal-dots' });
            const dotsToShow = Math.min(notes.length, 3);
            for (let i = 0; i < dotsToShow; i++) {
                dotContainer.createDiv({ cls: 'komo-cal-dot' });
            }
        }

        // Click handler - show notes popup
        cell.addEventListener('click', (e) => {
            if (notes.length > 0) {
                showNotesPopup(e, notes, d, calY, calM);
            } else {
                // Try to open daily note
                const ds = `${calY}-${String(calM+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
                const file = app.vault.getMarkdownFiles().find(f => f.basename === ds);
                if (file) {
                    app.workspace.openLinkText(file.path, '', false);
                } else if (isToday) {
                    try { app.commands.executeCommandById('daily-notes:goto-today'); }
                    catch(e) { new Notice(`No note for ${ds}`); }
                } else {
                    new Notice(`No daily note for ${ds}`);
                }
            }
        });
    }
}

function showNotesPopup(e, notes, day, year, month) {
    document.querySelector('.komo-cal-popup')?.remove();

    const popup = document.body.createDiv({ cls: 'komo-cal-popup' });
    popup.style.left = `${e.clientX}px`;
    popup.style.top = `${e.clientY + 20}px`;

    const header = popup.createDiv({ cls: 'komo-cal-popup-header' });
    const dateStr = `${year}-${String(month+1).padStart(2,'0')}-${String(day).padStart(2,'0')}`;
    header.textContent = dateStr;

    const list = popup.createDiv({ cls: 'komo-cal-popup-list' });
    if (notes.length > 3) {
        list.style.maxHeight = '120px';
        list.style.overflowY = 'auto';
    }

    notes.forEach(note => {
        const item = list.createDiv({ cls: 'komo-cal-popup-item' });
        item.createDiv({ cls: 'komo-cal-popup-dot' });
        item.createDiv({ cls: 'komo-cal-popup-name', text: note.name });
        item.addEventListener('click', () => {
            app.workspace.openLinkText(note.path, '', false);
            popup.remove();
        });
    });

    setTimeout(() => {
        document.addEventListener('click', function closePopup(e) {
            if (!popup.contains(e.target)) {
                popup.remove();
                document.removeEventListener('click', closePopup);
            }
        });
    }, 10);
}

renderCal();

// ── POMODORO TIMER ──────────────────────────
const pomodoroCard = colR.createDiv({ cls: 'komo-card komo-pomodoro-card' });
pomodoroCard.createDiv({ cls: 'komo-label', text: 'pomodoro' });

const pomodoroContainer = pomodoroCard.createDiv({ cls: 'komo-pomodoro' });

// Load settings
let pomoTime = parseInt(localStorage.getItem('komo-pomo-time') || '25');
let pomoBreak = parseInt(localStorage.getItem('komo-pomo-break') || '5');
let pomoCount = parseInt(localStorage.getItem('komo-pomo-count') || '0');
let pomoState = localStorage.getItem('komo-pomo-state') || 'idle'; // idle, running, paused, break
let pomoRemaining = parseInt(localStorage.getItem('komo-pomo-remaining') || (pomoTime * 60));
let pomoInterval = null;

function formatTime(seconds) {
    const m = Math.floor(seconds / 60);
    const s = seconds % 60;
    return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
}

function renderPomodoro() {
    pomodoroContainer.innerHTML = '';

    // Timer display
    const timerDisplay = pomodoroContainer.createDiv({ cls: 'komo-pomo-timer' });
    timerDisplay.textContent = formatTime(pomoRemaining);

    // Progress bar
    const totalTime = pomoState === 'break' ? (pomoBreak * 60) : (pomoTime * 60);
    const progress = ((totalTime - pomoRemaining) / totalTime) * 100;
    const progressBar = pomodoroContainer.createDiv({ cls: 'komo-pomo-progress' });
    const progressFill = progressBar.createDiv({ cls: 'komo-pomo-progress-fill' });
    progressFill.style.width = `${progress}%`;
    progressFill.style.background = pomoState === 'break' ? 'var(--komo-green)' : 'var(--komo-sakura)';

    // Status
    const statusText = pomodoroContainer.createDiv({ cls: 'komo-pomo-status' });
    if (pomoState === 'idle') statusText.textContent = 'ready to focus';
    else if (pomoState === 'running') statusText.textContent = 'focusing...';
    else if (pomoState === 'paused') statusText.textContent = 'paused';
    else if (pomoState === 'break') statusText.textContent = 'break time';

    // Controls
    const controls = pomodoroContainer.createDiv({ cls: 'komo-pomo-controls' });

    const startBtn = controls.createDiv({ cls: 'komo-pomo-btn komo-pomo-btn-primary', text: pomoState === 'running' ? '⏸' : '▶' });
    const resetBtn = controls.createDiv({ cls: 'komo-pomo-btn', text: '↺' });
    const settingsBtn = controls.createDiv({ cls: 'komo-pomo-btn', text: '⚙' });

    // Today's count
    const countDisplay = pomodoroContainer.createDiv({ cls: 'komo-pomo-count' });
    countDisplay.textContent = `today: ${pomoCount} pomodoros`;

    startBtn.addEventListener('click', () => {
        if (pomoState === 'running') {
            pausePomodoro();
        } else {
            startPomodoro();
        }
        renderPomodoro();
    });

    resetBtn.addEventListener('click', () => {
        resetPomodoro();
        renderPomodoro();
    });

    settingsBtn.addEventListener('click', showPomodoroSettings);
}

function startPomodoro() {
    pomoState = pomoState === 'break' ? 'break' : 'running';
    localStorage.setItem('komo-pomo-state', pomoState);

    pomoInterval = setInterval(() => {
        pomoRemaining--;
        localStorage.setItem('komo-pomo-remaining', pomoRemaining);

        if (pomoRemaining <= 0) {
            completePomodoro();
        }
        renderPomodoro();
    }, 1000);
}

function pausePomodoro() {
    pomoState = 'paused';
    localStorage.setItem('komo-pomo-state', pomoState);
    if (pomoInterval) {
        clearInterval(pomoInterval);
        pomoInterval = null;
    }
}

function resetPomodoro() {
    if (pomoInterval) {
        clearInterval(pomoInterval);
        pomoInterval = null;
    }
    pomoState = 'idle';
    pomoRemaining = pomoTime * 60;
    localStorage.setItem('komo-pomo-state', pomoState);
    localStorage.setItem('komo-pomo-remaining', pomoRemaining);
}

function completePomodoro() {
    if (pomoInterval) {
        clearInterval(pomoInterval);
        pomoInterval = null;
    }

    if (pomoState === 'break') {
        // Break finished, back to work
        pomoState = 'idle';
        pomoRemaining = pomoTime * 60;
        new Notice('Break finished! Ready to focus?');
    } else {
        // Work finished, start break
        pomoCount++;
        localStorage.setItem('komo-pomo-count', pomoCount);
        pomoState = 'break';
        pomoRemaining = pomoBreak * 60;
        new Notice('Pomodoro complete! Take a break 🌸');
    }

    localStorage.setItem('komo-pomo-state', pomoState);
    localStorage.setItem('komo-pomo-remaining', pomoRemaining);
}

function showPomodoroSettings() {
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal' });

    modal.createDiv({ cls: 'komo-modal-title', text: 'pomodoro settings' });

    const form = modal.createDiv({ cls: 'komo-card-form' });

    const workRow = form.createDiv({ cls: 'komo-form-row' });
    workRow.createEl('label', { text: 'work time (minutes)' });
    const workInput = workRow.createEl('input', {
        cls: 'komo-modal-search',
        type: 'number',
        attr: { value: pomoTime, min: '1', max: '60' }
    });

    const breakRow = form.createDiv({ cls: 'komo-form-row' });
    breakRow.createEl('label', { text: 'break time (minutes)' });
    const breakInput = breakRow.createEl('input', {
        cls: 'komo-modal-search',
        type: 'number',
        attr: { value: pomoBreak, min: '1', max: '30' }
    });

    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: 'save' });

    cancelBtn.addEventListener('click', () => overlay.remove());
    saveBtn.addEventListener('click', () => {
        pomoTime = parseInt(workInput.value) || 25;
        pomoBreak = parseInt(breakInput.value) || 5;
        localStorage.setItem('komo-pomo-time', pomoTime);
        localStorage.setItem('komo-pomo-break', pomoBreak);

        // Reset if idle
        if (pomoState === 'idle' || pomoState === 'paused') {
            pomoRemaining = pomoTime * 60;
            localStorage.setItem('komo-pomo-remaining', pomoRemaining);
        }

        renderPomodoro();
        overlay.remove();
    });

    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
}

// Resume if was running
if (pomoState === 'running') {
    startPomodoro();
}

renderPomodoro();
```

```dataviewjs
// Dashboard-Komorebi
// more dashboards → https://github.com/InlitX/Obsidian-Dashboard-Gallery

const wrap = dv.container.createDiv({ cls: 'komo-bottom-block' });

// Create bottom row with 2 columns
const bottomRow = wrap.createDiv({ cls: 'komo-bottom-row' });

// ══════════════════════════════════════════════
//  LEFT: HABIT TRACKER
// ══════════════════════════════════════════════
const habitCard = bottomRow.createDiv({ cls: 'komo-card komo-habit-card' });

// Header
const habitHdr = habitCard.createDiv({ cls: 'komo-habit-hdr' });
habitHdr.createDiv({ cls: 'komo-label', text: 'habits' });
const addBtn   = habitHdr.createDiv({ cls: 'komo-add-habit-btn', text: '+ add' });

let habits = JSON.parse(localStorage.getItem('komo-habits') || '[]');

const habitGrid = habitCard.createDiv({ cls: 'komo-habit-grid' });

// Week header row — rendered inside each column by renderHabits()

// Bio bar elements
const bioRow     = habitCard.createDiv({ cls: 'komo-bio-row' });
const bioLbl     = bioRow.createDiv({ cls: 'komo-bio-lbl', text: '○ starting' });
const bioBarWrap = bioRow.createDiv({ cls: 'komo-bio-bar-wrap' });
const bioBar     = bioBarWrap.createDiv({ cls: 'komo-bio-bar' });
const bioPct     = bioRow.createDiv({ cls: 'komo-bio-pct', text: '0%' });

function calcPct(habitName) {
    let cnt = 0;
    for (let d = 1; d <= 7; d++) {
        if (localStorage.getItem(`komo-h-${habitName}-d${d}`)) cnt++;
    }
    return { cnt, pct: Math.round((cnt / 7) * 100) };
}

function updateBioBar() {
    const total = habits.length * 7;
    let done = 0;
    habits.forEach(h => {
        for (let d = 1; d <= 7; d++) {
            if (localStorage.getItem(`komo-h-${h}-d${d}`)) done++;
        }
    });
    const pct = total > 0 ? Math.round((done / total) * 100) : 0;
    bioBar.style.width = `${pct}%`;
    bioPct.textContent = `${pct}%`;
    bioLbl.textContent = pct >= 80 ? '🌸 consistent' : pct >= 50 ? '⚡ building' : '○ starting';
}

function buildHabitColumn(container, habitSubset, globalIdxOffset, compact) {
    const rowCols = compact
        ? '68px repeat(7, 18px) 28px'
        : '130px repeat(7, 28px) 40px';
    const dotSize = compact ? '14px' : '20px';
    const nameFz  = compact ? '0.63rem' : '0.73rem';
    const pctFz   = compact ? '0.56rem' : '0.63rem';
    const dayFz   = compact ? '0.5rem'  : '0.58rem';

    // Header
    const hdr = container.createDiv({ cls: 'komo-week-hdr' });
    hdr.style.cssText = `display:grid;grid-template-columns:${rowCols};align-items:center;gap:${compact?'2px':'4px'};padding:0 4px 6px;border-bottom:1px solid var(--komo-border);margin-bottom:4px;`;
    hdr.createDiv({ cls: 'komo-week-hdr-name' });
    ['Mon','Tue','Wed','Thu','Fri','Sat','Sun'].forEach(d => {
        const dh = hdr.createDiv({ cls: 'komo-week-hdr-day', text: d });
        if (compact) dh.style.fontSize = dayFz;
    });
    hdr.createDiv({ cls: 'komo-week-hdr-pct', text: '%' });

    habitSubset.forEach((habit, localIdx) => {
        const idx = globalIdxOffset + localIdx;
        const row = container.createDiv({ cls: 'komo-habit-row' });
        row.style.cssText = `display:grid;grid-template-columns:${rowCols};align-items:center;gap:${compact?'2px':'4px'};padding:3px 4px;border-radius:3px;`;

        const nameEl = row.createEl('div', { cls: 'komo-habit-name', text: habit });
        nameEl.style.fontSize = nameFz;
        nameEl.title = 'right-click to rename / delete';
        nameEl.addEventListener('contextmenu', e => { e.preventDefault(); showCtxMenu(e, idx); });

        for (let d = 1; d <= 7; d++) {
            const key = `komo-h-${habit}-d${d}`;
            const dot = row.createDiv({ cls: `komo-dot ${localStorage.getItem(key) ? 'on' : ''}` });
            if (compact) { dot.style.width = dotSize; dot.style.height = dotSize; }
            dot.addEventListener('click', () => {
                dot.classList.toggle('on');
                if (dot.classList.contains('on')) localStorage.setItem(key, '1');
                else localStorage.removeItem(key);
                const { pct: np } = calcPct(habit);
                const rowPct = row.querySelector('.komo-habit-pct');
                if (rowPct) rowPct.textContent = `${np}%`;
                updateBioBar();
            });
        }

        const { pct: initPct } = calcPct(habit);
        const pctEl = row.createDiv({ cls: 'komo-habit-pct', text: `${initPct}%` });
        if (compact) pctEl.style.fontSize = pctFz;
    });
}

function renderHabits() {
    habitGrid.innerHTML = '';

    const twoCol = habits.length > 4;

    if (twoCol) {
        // Inline styles win over CSS naturally — no !important needed
        habitGrid.style.display = 'grid';
        habitGrid.style.gridTemplateColumns = '1fr 1fr';
        habitGrid.style.gap = '0 14px';
        habitGrid.style.alignItems = 'start';
        habitGrid.style.flexDirection = '';
        const mid  = Math.ceil(habits.length / 2);
        const colA = habitGrid.createDiv({ cls: 'komo-habit-col' });
        const colB = habitGrid.createDiv({ cls: 'komo-habit-col' });
        colA.style.cssText = 'display:flex;flex-direction:column;gap:2px;min-width:0;overflow:hidden;';
        colB.style.cssText = 'display:flex;flex-direction:column;gap:2px;min-width:0;overflow:hidden;';
        buildHabitColumn(colA, habits.slice(0, mid), 0, true);
        buildHabitColumn(colB, habits.slice(mid), mid, true);
    } else {
        habitGrid.style.display = 'flex';
        habitGrid.style.flexDirection = 'column';
        habitGrid.style.gridTemplateColumns = '';
        habitGrid.style.gap = '2px';
        habitGrid.style.alignItems = '';
        buildHabitColumn(habitGrid, habits, 0, false);
    }

    updateBioBar();
}

function showHabitModal({ title, defaultValue = '', confirmText, onConfirm }) {
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal' });
    modal.createDiv({ cls: 'komo-modal-title', text: title });
    const form = modal.createDiv({ cls: 'komo-card-form' });
    const row = form.createDiv({ cls: 'komo-form-row' });
    row.createEl('label', { text: 'name' });
    const input = row.createEl('input', {
        cls: 'komo-modal-search',
        attr: { placeholder: 'habit name...', type: 'text', value: defaultValue }
    });
    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn   = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: confirmText });
    const doSave = () => { const v = input.value.trim(); if (v) { onConfirm(v); } overlay.remove(); };
    cancelBtn.addEventListener('click', () => overlay.remove());
    saveBtn.addEventListener('click', doSave);
    input.addEventListener('keydown', e => { if (e.key === 'Enter') doSave(); });
    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
    setTimeout(() => input.focus(), 30);
}

function showCtxMenu(e, idx) {
    document.querySelector('.komo-ctx-menu')?.remove();
    const menu = document.body.createDiv({ cls: 'komo-ctx-menu' });
    menu.style.cssText = `left:${e.clientX}px;top:${e.clientY}px;`;

    const rename = menu.createDiv({ cls: 'komo-ctx-item', text: '✎  rename' });
    rename.addEventListener('click', () => {
        menu.remove();
        showHabitModal({
            title: 'rename habit',
            defaultValue: habits[idx],
            confirmText: 'rename',
            onConfirm: (newName) => {
                for (let d = 1; d <= 7; d++) {
                    const oldKey = `komo-h-${habits[idx]}-d${d}`;
                    const newKey = `komo-h-${newName}-d${d}`;
                    const val = localStorage.getItem(oldKey);
                    if (val) { localStorage.setItem(newKey, val); localStorage.removeItem(oldKey); }
                }
                habits[idx] = newName;
                localStorage.setItem('komo-habits', JSON.stringify(habits));
                renderHabits();
            }
        });
    });

    const del = menu.createDiv({ cls: 'komo-ctx-item komo-ctx-del', text: '✕  remove' });
    del.addEventListener('click', () => {
        for (let d = 1; d <= 7; d++) localStorage.removeItem(`komo-h-${habits[idx]}-d${d}`);
        habits.splice(idx, 1);
        localStorage.setItem('komo-habits', JSON.stringify(habits));
        renderHabits();
        menu.remove();
    });

    document.addEventListener('click', () => menu.remove(), { once: true });
}

addBtn.addEventListener('click', () => {
    showHabitModal({
        title: 'add habit',
        confirmText: 'add',
        onConfirm: (name) => {
            habits.push(name);
            localStorage.setItem('komo-habits', JSON.stringify(habits));
            renderHabits();
        }
    });
});

renderHabits();

// ══════════════════════════════════════════════
//  RIGHT: MUSIC PLAYER
// ══════════════════════════════════════════════
const musicCard = bottomRow.createDiv({ cls: 'komo-card komo-music-card' });


const musicPlayer = musicCard.createDiv({ cls: 'komo-music-player' });

// Settings button
const settingsBtn = musicCard.createDiv({ cls: 'komo-music-settings' });
try { setIcon(settingsBtn, 'settings'); } catch(e) { settingsBtn.textContent = '⚙'; }
settingsBtn.style.position = 'absolute';
settingsBtn.style.top = '14px';
settingsBtn.style.right = '14px';
settingsBtn.style.zIndex = '100';
settingsBtn.style.padding = '2px 5px';
settingsBtn.style.background = 'var(--komo-overlay)';
settingsBtn.style.border = '1px solid var(--komo-border)';
settingsBtn.style.borderRadius = '4px';
settingsBtn.style.cursor = 'pointer';
settingsBtn.style.opacity = '0.5';
settingsBtn.style.fontSize = '11px';
settingsBtn.addEventListener('click', showMusicSettings);

// Card height
musicCard.style.height = '260px';

// Load music settings
let musicMode = localStorage.getItem('komo-music-mode') || 'none';
let musicSource = localStorage.getItem('komo-music-source') || '';

function renderMusicPlayer() {
    musicPlayer.innerHTML = '';
    musicPlayer.classList.remove('komo-music-img-mode');

    if (musicMode === 'none') {
        musicPlayer.classList.add('komo-music-img-mode');
        if (musicSource) {
            const bgImage = musicPlayer.createDiv({ cls: 'komo-music-bg-image' });
            bgImage.style.cssText = 'width: 100%; height: 100%; background-size: cover; background-position: center; background-repeat: no-repeat; overflow: hidden; border-radius: 0;';
            
            if (musicSource.startsWith('http')) {
                bgImage.style.backgroundImage = `url('${musicSource}')`;
            } else {
                try {
                    const file = app.vault.getAbstractFileByPath(musicSource);
                    if (file) {
                        const url = app.vault.adapter.getResourcePath(musicSource);
                        bgImage.style.backgroundImage = `url('${url}')`;
                    }
                } catch(e) {}
            }
        } else {
            // No image configured — show placeholder
            const ph = musicPlayer.createDiv({ cls: 'komo-music-ph' });
            ph.createDiv({ cls: 'komo-music-ph-icon', text: '🎵' });
            ph.createDiv({ cls: 'komo-music-ph-hint', text: 'configure your music' });
        }
        return;
    }

    if (musicMode === 'spotify') {
        const savedPlaylists = JSON.parse(localStorage.getItem('komo-spotify-playlists') || '[]');
        
        if (musicSource) {
            const match = musicSource.match(/(playlist|album)\/([a-zA-Z0-9]+)/);
            const spotifyType = match ? match[1] : 'playlist';
            const spotifyId   = match ? match[2] : musicSource;

            const spotifyContainer = musicPlayer.createDiv({ cls: 'komo-spotify-container' });
            
            const iframe = spotifyContainer.createEl('iframe', {
                attr: {
                    src: `https://open.spotify.com/embed/${spotifyType}/${spotifyId}?theme=0`,
                    width: '100%',
                    height: '100%',
                    frameborder: '0',
                    allowtransparency: 'true',
                    allow: 'encrypted-media'
                }
            });
        } else {
            musicPlayer.createDiv({ cls: 'komo-music-error', text: '⚠ no playlist selected' });
        }
        
        if (savedPlaylists.length > 0) {
            const playlistWrap = musicPlayer.createDiv({ cls: 'komo-playlist-wrap' });
            
            savedPlaylists.slice(0, 4).forEach((pl, idx) => {
                const plBtn = playlistWrap.createDiv({ cls: 'komo-playlist-btn' });
                plBtn.textContent = pl.name.slice(0, 10);
                plBtn.title = pl.name;
                plBtn.addEventListener('click', () => {
                    musicSource = pl.url;
                    localStorage.setItem('komo-music-source', musicSource);
                    renderMusicPlayer();
                    new Notice('✅ Playing: ' + pl.name);
                });
            });
        }
    } else if (musicMode === 'local') {
        const localPlayer = musicPlayer.createDiv({ cls: 'komo-local-player' });

        if (musicSource) {
            try {
                const folder = app.vault.getAbstractFileByPath(musicSource);
                if (folder && folder.children) {
                    const audioFiles = folder.children.filter(f => 
                        f.name.toLowerCase().endsWith('.mp3') || 
                        f.name.toLowerCase().endsWith('.wav') ||
                        f.name.toLowerCase().endsWith('.ogg') ||
                        f.name.toLowerCase().endsWith('.m4a')
                    );
                    
                    if (audioFiles.length > 0) {
                        let currentIndex = 0;
                        let isPlaying = false;
                        
                        const audio = localPlayer.createEl('audio');
                        audio.style.display = 'none';
                        
                        // Track info
                        const infoRow = localPlayer.createDiv({ cls: 'komo-local-info' });
                        infoRow.createDiv({ cls: 'komo-local-icon', text: '♪' });
                        const trackMeta = infoRow.createDiv({ cls: 'komo-local-meta' });
                        const trackName = trackMeta.createDiv({ cls: 'komo-local-name' });
                        const trackTime = trackMeta.createDiv({ cls: 'komo-local-time', text: '0:00 / --:--' });
                        
                        // Progress bar
                        const progressBar = localPlayer.createDiv({ cls: 'komo-local-progress' });
                        const progressFill = progressBar.createDiv({ cls: 'komo-local-progress-fill' });
                        progressBar.addEventListener('click', (e) => {
                            if (audio.duration) {
                                const rect = progressBar.getBoundingClientRect();
                                audio.currentTime = ((e.clientX - rect.left) / rect.width) * audio.duration;
                            }
                        });
                        
                        // Controls
                        const controls = localPlayer.createDiv({ cls: 'komo-local-controls' });
                        const prevBtn = controls.createDiv({ cls: 'komo-local-ctrl', text: '⏮' });
                        const playBtn = controls.createDiv({ cls: 'komo-local-ctrl komo-local-ctrl-main', text: '▶' });
                        const nextBtn = controls.createDiv({ cls: 'komo-local-ctrl', text: '⏭' });
                        const counter = controls.createDiv({ cls: 'komo-local-counter', text: `1 / ${audioFiles.length}` });
                        
                        function fmtTime(s) {
                            const m = Math.floor(s / 60);
                            return `${m}:${String(Math.floor(s % 60)).padStart(2,'0')}`;
                        }
                        
                        audio.addEventListener('timeupdate', () => {
                            if (audio.duration) {
                                progressFill.style.width = `${(audio.currentTime / audio.duration) * 100}%`;
                                trackTime.textContent = `${fmtTime(audio.currentTime)} / ${fmtTime(audio.duration)}`;
                            }
                        });
                        
                        audio.addEventListener('ended', () => playTrack((currentIndex + 1) % audioFiles.length));
                        
                        const playTrack = (index) => {
                            currentIndex = index;
                            const file = audioFiles[index];
                            audio.src = app.vault.adapter.getResourcePath(file.path);
                            trackName.textContent = file.name.replace(/\.[^/.]+$/, '');
                            trackName.title = trackName.textContent;
                            trackTime.textContent = '0:00 / --:--';
                            progressFill.style.width = '0%';
                            counter.textContent = `${index + 1} / ${audioFiles.length}`;
                            audio.play();
                            playBtn.textContent = '⏸';
                            isPlaying = true;
                        };
                        
                        prevBtn.addEventListener('click', () => playTrack((currentIndex - 1 + audioFiles.length) % audioFiles.length));
                        nextBtn.addEventListener('click', () => playTrack((currentIndex + 1) % audioFiles.length));
                        playBtn.addEventListener('click', () => {
                            if (isPlaying) { audio.pause(); playBtn.textContent = '▶'; isPlaying = false; }
                            else { audio.play(); playBtn.textContent = '⏸'; isPlaying = true; }
                        });
                        
                        playTrack(0);
                    } else {
                        const err = localPlayer.createDiv({ cls: 'komo-local-empty' });
                        err.createDiv({ cls: 'komo-local-empty-icon', text: '⚠' });
                        err.createDiv({ cls: 'komo-local-empty-text', text: 'no audio files found' });
                    }
                } else {
                    const err = localPlayer.createDiv({ cls: 'komo-local-empty' });
                    err.createDiv({ cls: 'komo-local-empty-icon', text: '⚠' });
                    err.createDiv({ cls: 'komo-local-empty-text', text: 'folder not found' });
                }
            } catch(e) {
                const err = localPlayer.createDiv({ cls: 'komo-local-empty' });
                err.createDiv({ cls: 'komo-local-empty-text', text: '⚠ ' + e.message });
            }
        } else {
            const empty = localPlayer.createDiv({ cls: 'komo-local-empty' });
            empty.createDiv({ cls: 'komo-local-empty-icon', text: '🎧' });
            empty.createDiv({ cls: 'komo-local-empty-text', text: 'select a folder' });
        }
    }
}

function showMusicSettings() {
    const overlay = document.body.createDiv({ cls: 'komo-modal-overlay' });
    const modal = overlay.createDiv({ cls: 'komo-modal' });

    modal.createDiv({ cls: 'komo-modal-title', text: 'music settings' });

    const form = modal.createDiv({ cls: 'komo-card-form' });

    const modeRow = form.createDiv({ cls: 'komo-form-row' });
    modeRow.createEl('label', { text: 'source' });
    const modeSelect = modeRow.createEl('select', { cls: 'komo-modal-search' });
    [
        { value: 'none', label: 'disabled (image/gif)' },
        { value: 'spotify', label: 'spotify' },
        { value: 'local', label: 'local music' }
    ].forEach(opt => {
        const option = modeSelect.createEl('option', { text: opt.label, value: opt.value });
        if (opt.value === musicMode) option.selected = true;
    });

    // Source input row - label changes based on mode
    const sourceLabel = form.createDiv({ cls: 'komo-form-row' });
    const sourceLabelText = sourceLabel.createEl('label');
    
    const sourceInputRow = form.createDiv({ cls: 'komo-form-row' });
    sourceInputRow.style.display = 'flex';
    sourceInputRow.style.gap = '10px';
    
    // Inicializar con valor vacío - se llenará según el modo
    let initialSource = '';
    if (musicMode === 'spotify' || musicMode === 'local') {
        initialSource = musicSource;
    }
    
    const sourceInput = sourceInputRow.createEl('input', {
        cls: 'komo-modal-search',
        attr: {
            placeholder: '...',
            value: initialSource,
            style: 'flex: 1;'
        }
    });

    // Folder picker button - ONLY for local mode
    const folderPickerBtn = sourceInputRow.createEl('button', {
        cls: 'komo-modal-btn',
        text: '📁',
        attr: { style: 'padding: 8px 12px; cursor: pointer; display: none;' }
    });
    
    // Spotify playlists section
    const playlistsSection = form.createDiv({ cls: 'komo-playlists-section' });
    const addPlaylistSection = form.createDiv({ cls: 'komo-add-playlist-section' });
    
    // Saved playlists data
    let savedPlaylists = JSON.parse(localStorage.getItem('komo-spotify-playlists') || '[]');
    
    // Function to update UI based on selected mode
    function updateModeUI() {
        const currentMode = modeSelect.value;

        sourceInput.value = '';

        if (currentMode === 'spotify') {
            sourceLabelText.textContent = 'spotify url:';
            sourceInput.placeholder = 'https://open.spotify.com/playlist/ or /album/...';
            folderPickerBtn.style.display = 'none';
            playlistsSection.style.display = 'block';
            addPlaylistSection.style.display = 'block';
        } else if (currentMode === 'local') {
            sourceLabelText.textContent = 'music folder:';
            sourceInput.placeholder = 'Select a folder...';
            folderPickerBtn.style.display = 'block';
            playlistsSection.style.display = 'none';
            addPlaylistSection.style.display = 'none';
        } else {
            sourceLabelText.textContent = 'image/gif url:';
            sourceInput.placeholder = 'https://example.com/anim.gif or path/to/image.gif';
            folderPickerBtn.style.display = 'none';
            playlistsSection.style.display = 'none';
            addPlaylistSection.style.display = 'none';
        }
        
        renderSavedPlaylists();
    }
    
    // Initial setup
    updateModeUI();
    
    // Listen for mode changes - saves immediately
    modeSelect.addEventListener('change', () => {
        musicMode = modeSelect.value;
        localStorage.setItem('komo-music-mode', musicMode);
        updateModeUI();
    });
    
    // Function to render saved playlists
    function renderSavedPlaylists() {
        playlistsSection.innerHTML = '';
        addPlaylistSection.innerHTML = '';
        
        if (modeSelect.value !== 'spotify') return;
        
        // Saved playlists list
        if (savedPlaylists.length > 0) {
            const plLabel = playlistsSection.createEl('label', { text: 'saved playlists' });
            plLabel.style.display = 'block';
            plLabel.style.marginBottom = '8px';
            
            const playlistsContainer = playlistsSection.createDiv({ cls: 'komo-playlists-container' });
            playlistsContainer.style.cssText = 'max-height: 150px; overflow-y: auto; border: 1px solid var(--komo-border); border-radius: 4px; padding: 8px; margin-bottom: 15px;';
            
            savedPlaylists.forEach((pl, idx) => {
                const plItem = playlistsContainer.createDiv({ cls: 'komo-playlist-item' });
                plItem.style.cssText = 'display: flex; justify-content: space-between; align-items: center; padding: 2px 6px; margin-bottom: 2px; background: var(--komo-bg); border-radius: 4px; cursor: pointer; font-size: 0.7rem;';
                plItem.addEventListener('click', () => {
                    sourceInput.value = pl.url;
                    musicSource = pl.url;
                    localStorage.setItem('komo-music-source', musicSource);
                    new Notice('✅ Playlist loaded: ' + pl.name);
                });
                
                const plName = plItem.createDiv({ text: pl.name });
                plName.style.cssText = 'flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;';
                
                const plDelete = plItem.createEl('span', { text: '✕' });
                plDelete.style.cssText = 'cursor: pointer; color: #f38ba8; padding: 4px;';
                plDelete.addEventListener('click', (e) => {
                    e.stopPropagation();
                    savedPlaylists.splice(idx, 1);
                    localStorage.setItem('komo-spotify-playlists', JSON.stringify(savedPlaylists));
                    renderSavedPlaylists();
                });
            });
        }
        
        // Add new playlist inputs
        const newPlLabel = addPlaylistSection.createEl('label', { text: 'add new playlist' });
        newPlLabel.style.display = 'block';
        newPlLabel.style.marginBottom = '8px';
        
        const newPlInput = addPlaylistSection.createEl('input', {
            cls: 'komo-modal-search',
            attr: { placeholder: 'Playlist name', style: 'margin-bottom: 8px;' }
        });
        
        const newPlUrl = addPlaylistSection.createEl('input', {
            cls: 'komo-modal-search',
            attr: { placeholder: 'Spotify playlist URL', style: 'margin-bottom: 8px;' }
        });
        
        const addPlBtn = addPlaylistSection.createEl('button', {
            cls: 'komo-modal-btn komo-btn-save',
            text: '+ Add'
        });
        
        addPlBtn.addEventListener('click', () => {
            const name = newPlInput.value.trim();
            const url = newPlUrl.value.trim();
            
            if (!name || !url) {
                new Notice('⚠️ Enter name and URL');
                return;
            }
            
            if (!url.includes('spotify.com/')) {
                new Notice('❌ Invalid Spotify URL');
                return;
            }
            
            if (savedPlaylists.length >= 4) {
                new Notice('⚠️ Max 4 playlists');
                return;
            }
            
            savedPlaylists.push({ name: name.slice(0, 10), url });
            localStorage.setItem('komo-spotify-playlists', JSON.stringify(savedPlaylists));
            new Notice('✅ Playlist saved!');
            newPlInput.value = '';
            newPlUrl.value = '';
            renderSavedPlaylists();
        });
    }

    // Folder picker handler
    folderPickerBtn.addEventListener('click', async () => {
        // Open Obsidian's file picker
        const folders = app.vault.getAllFolders();
        
        // Create a simple folder selection modal
        const folderModal = document.body.createDiv({ cls: 'komo-modal-overlay' });
        const folderContent = folderModal.createDiv({ cls: 'komo-modal' });
        folderContent.style.maxWidth = '400px';
        
        folderContent.createDiv({ cls: 'komo-modal-title', text: 'select music folder' });
        
        const folderList = folderContent.createDiv({ cls: 'komo-folder-list' });
        folderList.style.cssText = 'max-height: 300px; overflow-y: auto; margin: 10px 0;';
        
        // Add root option
        const rootItem = folderList.createDiv({ cls: 'komo-folder-item' });
        rootItem.style.cssText = 'padding: 10px; cursor: pointer; border-bottom: 1px solid var(--komo-border);';
        rootItem.textContent = '📁 Vault Root';
        rootItem.addEventListener('click', () => {
            sourceInput.value = '/';
            folderModal.remove();
        });
        
        // Add all folders
        folders.forEach(folder => {
            const item = folderList.createDiv({ cls: 'komo-folder-item' });
            item.style.cssText = 'padding: 10px; cursor: pointer; border-bottom: 1px solid var(--komo-border);';
            item.textContent = '📁 ' + folder.name;
            item.addEventListener('click', () => {
                sourceInput.value = folder.path;
                folderModal.remove();
            });
            item.addEventListener('mouseover', () => item.style.background = 'var(--komo-hover)');
            item.addEventListener('mouseout', () => item.style.background = 'transparent');
        });
        
        const closeBtn = folderContent.createEl('button', {
            cls: 'komo-modal-btn komo-btn-cancel',
            text: 'cancel'
        });
        closeBtn.style.marginTop = '10px';
        closeBtn.addEventListener('click', () => folderModal.remove());
        
        folderModal.addEventListener('click', e => { if (e.target === folderModal) folderModal.remove(); });
    });

    const btns = modal.createDiv({ cls: 'komo-modal-btns' });
    const cancelBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-cancel', text: 'cancel' });
    const saveBtn = btns.createEl('button', { cls: 'komo-modal-btn komo-btn-save', text: 'save' });

    cancelBtn.addEventListener('click', () => overlay.remove());
    saveBtn.addEventListener('click', () => {
        musicMode = modeSelect.value;
        musicSource = sourceInput.value.trim();
        localStorage.setItem('komo-music-mode', musicMode);
        localStorage.setItem('komo-music-source', musicSource);
        renderMusicPlayer();
        overlay.remove();
    });

    overlay.addEventListener('click', e => { if (e.target === overlay) overlay.remove(); });
}

renderMusicPlayer();
```
