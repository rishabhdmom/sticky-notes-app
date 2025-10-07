<!doctype html>
<html lang="en">

<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Sticky Notes — Creative</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@400;500;600&display=swap');

    :root {
      --bg: #f3f4f6;
      --panel: #ffffff;
      --accent: #6366f1;
      --muted: #6b7280;
      --note-width: 240px;
      --note-height: 200px;
      --ui-radius: 12px;
    }

    html,
    body {
      height: 100%;
      margin: 0;
      font-family: 'Quicksand', sans-serif;
      color: #111
    }

    body {
      background: linear-gradient(135deg, #e0e7ff, #fdf2f8);
      padding: 18px;
      box-sizing: border-box
    }

    .topbar {
      display: flex;
      gap: 10px;
      align-items: center;
      margin-bottom: 12px
    }

    .title {
      font-weight: 600;
      font-size: 20px
    }

    .controls {
      margin-left: auto;
      display: flex;
      gap: 8px
    }

    button,
    .btn {
      background: var(--panel);
      border: 1px solid rgba(0, 0, 0, 0.06);
      padding: 8px 10px;
      border-radius: 8px;
      cursor: pointer;
      font-family: 'Quicksand', sans-serif
    }

    button:hover {
      box-shadow: 0 4px 12px rgba(16, 24, 40, 0.08)
    }

    .search {
      padding: 8px;
      border-radius: 8px;
      border: 1px solid rgba(0, 0, 0, 0.06);
      min-width: 180px;
      font-family: 'Quicksand', sans-serif
    }

    #board {
      position: relative;
      width: 100%;
      height: calc(100vh - 140px);
      overflow: auto;
      border-radius: 12px;
      padding: 14px;
    }

    .note {
      position: absolute;
      width: var(--note-width);
      height: var(--note-height);
      box-shadow: 0 10px 20px rgba(0, 0, 0, 0.15);
      border-radius: 18px;
      display: flex;
      flex-direction: column;
      user-select: none;
      overflow: hidden;
    }

    .note .header {
      display: flex;
      align-items: center;
      padding: 8px;
      border-top-left-radius: 18px;
      border-top-right-radius: 18px;
      gap: 8px;
      cursor: grab;
      background: rgba(0, 0, 0, 0.05)
    }

    .note .titlebar {
      flex: 1;
      font-weight: 600;
      font-size: 14px
    }

    .note .toolbar {
      display: flex;
      gap: 6px
    }

    .content {
      flex: 1;
      padding: 12px;
      overflow: auto;
      outline: none;
      background: transparent;
      font-size: 14px;
      line-height: 1.4
    }

    .content[contenteditable]:empty:before {
      content: 'Type here...';
      color: rgba(0, 0, 0, 0.22)
    }

    .resize-handle {
      position: absolute;
      width: 14px;
      height: 14px;
      right: 6px;
      bottom: 6px;
      cursor: se-resize;
      border-radius: 4px;
      background: rgba(0, 0, 0, 0.2)
    }

    .palette {
      display: flex;
      gap: 6px
    }

    .palette .sw {
      width: 18px;
      height: 18px;
      border-radius: 4px;
      border: 1px solid rgba(0, 0, 0, 0.06);
      cursor: pointer
    }

    /* Professional color palettes */
    .c-yellow {
      background: #fffbe6
    }

    .c-yellow .header {
      background: #fde68a
    }

    .c-pink {
      background: #fff0f6
    }

    .c-pink .header {
      background: #f9a8d4
    }

    .c-green {
      background: #ecfdf5
    }

    .c-green .header {
      background: #6ee7b7
    }

    .c-blue {
      background: #eff6ff
    }

    .c-blue .header {
      background: #93c5fd
    }

    .c-purple {
      background: #f5f3ff
    }

    .c-purple .header {
      background: #c4b5fd
    }

    @media (max-width:600px) {
      .title {
        font-size: 16px
      }

      :root {
        --note-width: 200px;
        --note-height: 160px
      }
    }

    .font-controls {
      margin-left: 12px;
    }

    .font-controls select {
      padding: 6px;
      border-radius: 6px;
      border: 1px solid #ccc;
      font-family: 'Quicksand', sans-serif
    }
  </style>
</head>

<body>
  <div class="topbar">
    <div class="title">Creative Sticky Notes</div>
    <input id="search" placeholder="Search notes..." class="search" />
    <div class="font-controls">
      <label for="fontSize">Font Size:</label>
      <select id="fontSize">
        <option value="12">Small</option>
        <option value="14" selected>Normal</option>
        <option value="16">Large</option>
        <option value="20">Extra Large</option>
      </select>
    </div>
    <div class="controls">
      <button id="newNote">+ New Note</button>
      <button id="export" class="small">Export</button>
      <button id="importBtn" class="small">Import</button>
      <button id="clearAll" class="small">Clear All</button>
    </div>
  </div>

  <div id="board" aria-label="notes board"></div>

  <input id="importFile" type="file" accept="application/json" style="display:none" />

  <script>
    const STORAGE_KEY = 'sticky_notes_v2';
    const board = document.getElementById('board');
    const newNoteBtn = document.getElementById('newNote');
    const clearAllBtn = document.getElementById('clearAll');
    const exportBtn = document.getElementById('export');
    const importBtn = document.getElementById('importBtn');
    const importFile = document.getElementById('importFile');
    const searchInput = document.getElementById('search');
    const fontSizeSelect = document.getElementById('fontSize');

    let notes = [];
    let zIndexCounter = 1;

    function uid() { return 'n_' + Date.now().toString(36) + '_' + Math.random().toString(36).slice(2, 7) }

    function save() { localStorage.setItem(STORAGE_KEY, JSON.stringify({ notes, zIndexCounter })); }
    function load() {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return;
      const data = JSON.parse(raw);
      notes = data.notes || [];
      zIndexCounter = data.zIndexCounter || 1;
    }

    function createNote(opts = {}) {
      const n = Object.assign({
        id: uid(),
        x: 60 + Math.random() * 100,
        y: 60 + Math.random() * 100,
        w: 240,
        h: 200,
        color: 'c-yellow',
        content: '',
        z: ++zIndexCounter,
        minimized: false,
        fontSize: 14
      }, opts);
      notes.push(n);
      renderNote(n);
      save();
      return n;
    }

    function removeNote(id) {
      const el = document.querySelector(`[data-id="${id}"]`);
      if (el) el.remove();
      notes = notes.filter(n => n.id !== id);
      save();
    }

    function clearBoard() {
      if (!confirm('Delete all notes? This cannot be undone.')) return;
      notes = []; board.innerHTML = ''; save();
    }

    function renderAll() {
      board.innerHTML = '';
      notes.forEach(renderNote);
    }

    function renderNote(n) {
      if (document.querySelector(`[data-id="${n.id}"]`)) return updateNoteElement(n);
      const el = document.createElement('div');
      el.className = 'note ' + (n.color || 'c-yellow');
      el.dataset.id = n.id;
      el.style.left = n.x + 'px';
      el.style.top = n.y + 'px';
      el.style.width = n.w + 'px';
      el.style.height = n.h + 'px';
      el.style.zIndex = n.z || (++zIndexCounter);

      const header = document.createElement('div'); header.className = 'header';
      const titlebar = document.createElement('div'); titlebar.className = 'titlebar'; titlebar.textContent = 'Note';
      const toolbar = document.createElement('div'); toolbar.className = 'toolbar';

      const palette = document.createElement('div'); palette.className = 'palette';
      ['c-yellow', 'c-pink', 'c-green', 'c-blue', 'c-purple'].forEach(c => {
        const sw = document.createElement('div'); sw.className = 'sw ' + c; sw.title = c.replace('c-', '');
        sw.addEventListener('click', () => { n.color = c; el.className = 'note ' + c; save(); });
        palette.appendChild(sw);
      });

      const minBtn = document.createElement('button'); minBtn.className = 'btn small'; minBtn.innerHTML = '—';
      minBtn.addEventListener('click', () => { n.minimized = !n.minimized; updateNoteElement(n); save(); });

      const delBtn = document.createElement('button'); delBtn.className = 'btn small'; delBtn.innerHTML = '✕';
      delBtn.addEventListener('click', () => { if (confirm('Delete this note?')) removeNote(n.id); });

      toolbar.appendChild(palette);
      toolbar.appendChild(minBtn);
      toolbar.appendChild(delBtn);

      header.appendChild(titlebar);
      header.appendChild(toolbar);
      el.appendChild(header);

      const content = document.createElement('div');
      content.className = 'content';
      content.contentEditable = true;
      content.spellcheck = false;
      content.innerHTML = n.content || '';
      content.style.fontSize = (n.fontSize || 14) + 'px';
      content.addEventListener('input', () => { n.content = content.innerHTML; save(); });
      content.addEventListener('focus', () => { bringToFront(el, n); });
      el.appendChild(content);

      const resize = document.createElement('div'); resize.className = 'resize-handle'; el.appendChild(resize);
      board.appendChild(el);

      header.addEventListener('mousedown', (e) => startDrag(e, el, n));
      header.addEventListener('touchstart', (e) => startDrag(e, el, n), { passive: false });
      resize.addEventListener('mousedown', (e) => startResize(e, el, n));
      resize.addEventListener('touchstart', (e) => startResize(e, el, n), { passive: false });
      el.addEventListener('mousedown', () => { bringToFront(el, n); });

      updateNoteElement(n);
    }

    function updateNoteElement(n) {
      const el = document.querySelector(`[data-id="${n.id}"]`);
      if (!el) return;
      el.className = 'note ' + (n.color || 'c-yellow');
      el.style.left = n.x + 'px'; el.style.top = n.y + 'px';
      el.style.width = n.w + 'px'; el.style.height = n.h + 'px';
      el.style.zIndex = n.z || (++zIndexCounter);
      const content = el.querySelector('.content');
      if (content) {
        content.innerHTML = n.content || '';
        content.style.display = n.minimized ? 'none' : '';
        content.style.fontSize = (n.fontSize || 14) + 'px';
      }
    }

    function bringToFront(el, n) { n.z = ++zIndexCounter; el.style.zIndex = n.z; save(); }

    function startDrag(e, el, n) {
      e.preventDefault();
      bringToFront(el, n);
      const startX = (e.touches ? e.touches[0].clientX : e.clientX);
      const startY = (e.touches ? e.touches[0].clientY : e.clientY);
      const origX = n.x, origY = n.y;
      const onMove = (ev) => {
        const mx = (ev.touches ? ev.touches[0].clientX : ev.clientX);
        const my = (ev.touches ? ev.touches[0].clientY : ev.clientY);
        n.x = origX + (mx - startX);
        n.y = origY + (my - startY);
        el.style.left = n.x + 'px'; el.style.top = n.y + 'px';
      };
      const onUp = () => { document.removeEventListener('mousemove', onMove); document.removeEventListener('mouseup', onUp); document.removeEventListener('touchmove', onMove); document.removeEventListener('touchend', onUp); save(); };
      document.addEventListener('mousemove', onMove); document.addEventListener('mouseup', onUp);
      document.addEventListener('touchmove', onMove, { passive: false }); document.addEventListener('touchend', onUp);
    }

    function startResize(e, el, n) {
      e.preventDefault(); e.stopPropagation();
      bringToFront(el, n);
      const startX = (e.touches ? e.touches[0].clientX : e.clientX);
      const startY = (e.touches ? e.touches[0].clientY : e.clientY);
      const origW = n.w, origH = n.h;
      const onMove = (ev) => {
        const mx = (ev.touches ? ev.touches[0].clientX : ev.clientX);
        const my = (ev.touches ? ev.touches[0].clientY : ev.clientY);
        n.w = Math.max(150, origW + (mx - startX));
        n.h = Math.max(100, origH + (my - startY));
        el.style.width = n.w + 'px'; el.style.height = n.h + 'px';
      };
      const onUp = () => { document.removeEventListener('mousemove', onMove); document.removeEventListener('mouseup', onUp); document.removeEventListener('touchmove', onMove); document.removeEventListener('touchend', onUp); save(); };
      document.addEventListener('mousemove', onMove); document.addEventListener('mouseup', onUp);
      document.addEventListener('touchmove', onMove, { passive: false }); document.addEventListener('touchend', onUp);
    }

    function exportNotes() {
      const dataStr = JSON.stringify({ notes, zIndexCounter }, null, 2);
      const blob = new Blob([dataStr], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url; a.download = 'sticky-notes.json'; a.click();
      URL.revokeObjectURL(url);
    }

    function importNotesFile(file) {
      const reader = new FileReader();
      reader.onload = () => {
        try {
          const parsed = JSON.parse(reader.result);
          if (Array.isArray(parsed.notes)) {
            parsed.notes.forEach(n => { n.id = n.id || uid(); notes.push(n); });
            zIndexCounter = Math.max(zIndexCounter, parsed.zIndexCounter || 1);
            renderAll(); save();
            alert('Imported ' + parsed.notes.length + ' notes');
          } else alert('Invalid file format');
        } catch (e) { alert('Import failed: ' + e.message) }
      };
      reader.readAsText(file);
    }

    newNoteBtn.addEventListener('click', () => createNote());
    clearAllBtn.addEventListener('click', clearBoard);
    exportBtn.addEventListener('click', exportNotes);
    importBtn.addEventListener('click', () => importFile.click());
    importFile.addEventListener('change', (ev) => { const f = ev.target.files[0]; if (f) importNotesFile(f); importFile.value = ''; });
    searchInput.addEventListener('input', () => {
      const q = searchInput.value.trim().toLowerCase();
      notes.forEach(n => {
        const el = document.querySelector(`[data-id="${n.id}"]`);
        if (!el) return;
        el.style.display = !q || (n.content || '').toLowerCase().includes(q) ? '' : 'none';
      });
    });

    fontSizeSelect.addEventListener('change', () => {
      const size = parseInt(fontSizeSelect.value);
      notes.forEach(n => { n.fontSize = size; updateNoteElement(n); });
      save();
    });

    load(); renderAll();
    if (notes.length === 0) {
      createNote({ content: 'Welcome! Drag the header to move. Resize with the corner. Change color & font size with controls.' });
    }

    window.addEventListener('beforeunload', save);
  </script>
</body>

</html>
<!doctype html>
<html lang="en">

<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Sticky Notes — Creative</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@400;500;600&display=swap');

    :root {
      --bg: #f3f4f6;
      --panel: #ffffff;
      --accent: #6366f1;
      --muted: #6b7280;
      --note-width: 240px;
      --note-height: 200px;
      --ui-radius: 12px;
    }

    html,
    body {
      height: 100%;
      margin: 0;
      font-family: 'Quicksand', sans-serif;
      color: #111
    }

    body {
      background: linear-gradient(135deg, #e0e7ff, #fdf2f8);
      padding: 18px;
      box-sizing: border-box
    }

    .topbar {
      display: flex;
      gap: 10px;
      align-items: center;
      margin-bottom: 12px
    }

    .title {
      font-weight: 600;
      font-size: 20px
    }

    .controls {
      margin-left: auto;
      display: flex;
      gap: 8px
    }

    button,
    .btn {
      background: var(--panel);
      border: 1px solid rgba(0, 0, 0, 0.06);
      padding: 8px 10px;
      border-radius: 8px;
      cursor: pointer;
      font-family: 'Quicksand', sans-serif
    }

    button:hover {
      box-shadow: 0 4px 12px rgba(16, 24, 40, 0.08)
    }

    .search {
      padding: 8px;
      border-radius: 8px;
      border: 1px solid rgba(0, 0, 0, 0.06);
      min-width: 180px;
      font-family: 'Quicksand', sans-serif
    }

    #board {
      position: relative;
      width: 100%;
      height: calc(100vh - 140px);
      overflow: auto;
      border-radius: 12px;
      padding: 14px;
    }

    .note {
      position: absolute;
      width: var(--note-width);
      height: var(--note-height);
      box-shadow: 0 10px 20px rgba(0, 0, 0, 0.15);
      border-radius: 18px;
      display: flex;
      flex-direction: column;
      user-select: none;
      overflow: hidden;
    }

    .note .header {
      display: flex;
      align-items: center;
      padding: 8px;
      border-top-left-radius: 18px;
      border-top-right-radius: 18px;
      gap: 8px;
      cursor: grab;
      background: rgba(0, 0, 0, 0.05)
    }

    .note .titlebar {
      flex: 1;
      font-weight: 600;
      font-size: 14px
    }

    .note .toolbar {
      display: flex;
      gap: 6px
    }

    .content {
      flex: 1;
      padding: 12px;
      overflow: auto;
      outline: none;
      background: transparent;
      font-size: 14px;
      line-height: 1.4
    }

    .content[contenteditable]:empty:before {
      content: 'Type here...';
      color: rgba(0, 0, 0, 0.22)
    }

    .resize-handle {
      position: absolute;
      width: 14px;
      height: 14px;
      right: 6px;
      bottom: 6px;
      cursor: se-resize;
      border-radius: 4px;
      background: rgba(0, 0, 0, 0.2)
    }

    .palette {
      display: flex;
      gap: 6px
    }

    .palette .sw {
      width: 18px;
      height: 18px;
      border-radius: 4px;
      border: 1px solid rgba(0, 0, 0, 0.06);
      cursor: pointer
    }

    /* Professional color palettes */
    .c-yellow {
      background: #fffbe6
    }

    .c-yellow .header {
      background: #fde68a
    }

    .c-pink {
      background: #fff0f6
    }

    .c-pink .header {
      background: #f9a8d4
    }

    .c-green {
      background: #ecfdf5
    }

    .c-green .header {
      background: #6ee7b7
    }

    .c-blue {
      background: #eff6ff
    }

    .c-blue .header {
      background: #93c5fd
    }

    .c-purple {
      background: #f5f3ff
    }

    .c-purple .header {
      background: #c4b5fd
    }

    @media (max-width:600px) {
      .title {
        font-size: 16px
      }

      :root {
        --note-width: 200px;
        --note-height: 160px
      }
    }

    .font-controls {
      margin-left: 12px;
    }

    .font-controls select {
      padding: 6px;
      border-radius: 6px;
      border: 1px solid #ccc;
      font-family: 'Quicksand', sans-serif
    }
  </style>
</head>

<body>
  <div class="topbar">
    <div class="title">Creative Sticky Notes</div>
    <input id="search" placeholder="Search notes..." class="search" />
    <div class="font-controls">
      <label for="fontSize">Font Size:</label>
      <select id="fontSize">
        <option value="12">Small</option>
        <option value="14" selected>Normal</option>
        <option value="16">Large</option>
        <option value="20">Extra Large</option>
      </select>
    </div>
    <div class="controls">
      <button id="newNote">+ New Note</button>
      <button id="export" class="small">Export</button>
      <button id="importBtn" class="small">Import</button>
      <button id="clearAll" class="small">Clear All</button>
    </div>
  </div>

  <div id="board" aria-label="notes board"></div>

  <input id="importFile" type="file" accept="application/json" style="display:none" />

  <script>
    const STORAGE_KEY = 'sticky_notes_v2';
    const board = document.getElementById('board');
    const newNoteBtn = document.getElementById('newNote');
    const clearAllBtn = document.getElementById('clearAll');
    const exportBtn = document.getElementById('export');
    const importBtn = document.getElementById('importBtn');
    const importFile = document.getElementById('importFile');
    const searchInput = document.getElementById('search');
    const fontSizeSelect = document.getElementById('fontSize');

    let notes = [];
    let zIndexCounter = 1;

    function uid() { return 'n_' + Date.now().toString(36) + '_' + Math.random().toString(36).slice(2, 7) }

    function save() { localStorage.setItem(STORAGE_KEY, JSON.stringify({ notes, zIndexCounter })); }
    function load() {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return;
      const data = JSON.parse(raw);
      notes = data.notes || [];
      zIndexCounter = data.zIndexCounter || 1;
    }

    function createNote(opts = {}) {
      const n = Object.assign({
        id: uid(),
        x: 60 + Math.random() * 100,
        y: 60 + Math.random() * 100,
        w: 240,
        h: 200,
        color: 'c-yellow',
        content: '',
        z: ++zIndexCounter,
        minimized: false,
        fontSize: 14
      }, opts);
      notes.push(n);
      renderNote(n);
      save();
      return n;
    }

    function removeNote(id) {
      const el = document.querySelector(`[data-id="${id}"]`);
      if (el) el.remove();
      notes = notes.filter(n => n.id !== id);
      save();
    }

    function clearBoard() {
      if (!confirm('Delete all notes? This cannot be undone.')) return;
      notes = []; board.innerHTML = ''; save();
    }

    function renderAll() {
      board.innerHTML = '';
      notes.forEach(renderNote);
    }

    function renderNote(n) {
      if (document.querySelector(`[data-id="${n.id}"]`)) return updateNoteElement(n);
      const el = document.createElement('div');
      el.className = 'note ' + (n.color || 'c-yellow');
      el.dataset.id = n.id;
      el.style.left = n.x + 'px';
      el.style.top = n.y + 'px';
      el.style.width = n.w + 'px';
      el.style.height = n.h + 'px';
      el.style.zIndex = n.z || (++zIndexCounter);

      const header = document.createElement('div'); header.className = 'header';
      const titlebar = document.createElement('div'); titlebar.className = 'titlebar'; titlebar.textContent = 'Note';
      const toolbar = document.createElement('div'); toolbar.className = 'toolbar';

      const palette = document.createElement('div'); palette.className = 'palette';
      ['c-yellow', 'c-pink', 'c-green', 'c-blue', 'c-purple'].forEach(c => {
        const sw = document.createElement('div'); sw.className = 'sw ' + c; sw.title = c.replace('c-', '');
        sw.addEventListener('click', () => { n.color = c; el.className = 'note ' + c; save(); });
        palette.appendChild(sw);
      });

      const minBtn = document.createElement('button'); minBtn.className = 'btn small'; minBtn.innerHTML = '—';
      minBtn.addEventListener('click', () => { n.minimized = !n.minimized; updateNoteElement(n); save(); });

      const delBtn = document.createElement('button'); delBtn.className = 'btn small'; delBtn.innerHTML = '✕';
      delBtn.addEventListener('click', () => { if (confirm('Delete this note?')) removeNote(n.id); });

      toolbar.appendChild(palette);
      toolbar.appendChild(minBtn);
      toolbar.appendChild(delBtn);

      header.appendChild(titlebar);
      header.appendChild(toolbar);
      el.appendChild(header);

      const content = document.createElement('div');
      content.className = 'content';
      content.contentEditable = true;
      content.spellcheck = false;
      content.innerHTML = n.content || '';
      content.style.fontSize = (n.fontSize || 14) + 'px';
      content.addEventListener('input', () => { n.content = content.innerHTML; save(); });
      content.addEventListener('focus', () => { bringToFront(el, n); });
      el.appendChild(content);

      const resize = document.createElement('div'); resize.className = 'resize-handle'; el.appendChild(resize);
      board.appendChild(el);

      header.addEventListener('mousedown', (e) => startDrag(e, el, n));
      header.addEventListener('touchstart', (e) => startDrag(e, el, n), { passive: false });
      resize.addEventListener('mousedown', (e) => startResize(e, el, n));
      resize.addEventListener('touchstart', (e) => startResize(e, el, n), { passive: false });
      el.addEventListener('mousedown', () => { bringToFront(el, n); });

      updateNoteElement(n);
    }

    function updateNoteElement(n) {
      const el = document.querySelector(`[data-id="${n.id}"]`);
      if (!el) return;
      el.className = 'note ' + (n.color || 'c-yellow');
      el.style.left = n.x + 'px'; el.style.top = n.y + 'px';
      el.style.width = n.w + 'px'; el.style.height = n.h + 'px';
      el.style.zIndex = n.z || (++zIndexCounter);
      const content = el.querySelector('.content');
      if (content) {
        content.innerHTML = n.content || '';
        content.style.display = n.minimized ? 'none' : '';
        content.style.fontSize = (n.fontSize || 14) + 'px';
      }
    }

    function bringToFront(el, n) { n.z = ++zIndexCounter; el.style.zIndex = n.z; save(); }

    function startDrag(e, el, n) {
      e.preventDefault();
      bringToFront(el, n);
      const startX = (e.touches ? e.touches[0].clientX : e.clientX);
      const startY = (e.touches ? e.touches[0].clientY : e.clientY);
      const origX = n.x, origY = n.y;
      const onMove = (ev) => {
        const mx = (ev.touches ? ev.touches[0].clientX : ev.clientX);
        const my = (ev.touches ? ev.touches[0].clientY : ev.clientY);
        n.x = origX + (mx - startX);
        n.y = origY + (my - startY);
        el.style.left = n.x + 'px'; el.style.top = n.y + 'px';
      };
      const onUp = () => { document.removeEventListener('mousemove', onMove); document.removeEventListener('mouseup', onUp); document.removeEventListener('touchmove', onMove); document.removeEventListener('touchend', onUp); save(); };
      document.addEventListener('mousemove', onMove); document.addEventListener('mouseup', onUp);
      document.addEventListener('touchmove', onMove, { passive: false }); document.addEventListener('touchend', onUp);
    }

    function startResize(e, el, n) {
      e.preventDefault(); e.stopPropagation();
      bringToFront(el, n);
      const startX = (e.touches ? e.touches[0].clientX : e.clientX);
      const startY = (e.touches ? e.touches[0].clientY : e.clientY);
      const origW = n.w, origH = n.h;
      const onMove = (ev) => {
        const mx = (ev.touches ? ev.touches[0].clientX : ev.clientX);
        const my = (ev.touches ? ev.touches[0].clientY : ev.clientY);
        n.w = Math.max(150, origW + (mx - startX));
        n.h = Math.max(100, origH + (my - startY));
        el.style.width = n.w + 'px'; el.style.height = n.h + 'px';
      };
      const onUp = () => { document.removeEventListener('mousemove', onMove); document.removeEventListener('mouseup', onUp); document.removeEventListener('touchmove', onMove); document.removeEventListener('touchend', onUp); save(); };
      document.addEventListener('mousemove', onMove); document.addEventListener('mouseup', onUp);
      document.addEventListener('touchmove', onMove, { passive: false }); document.addEventListener('touchend', onUp);
    }

    function exportNotes() {
      const dataStr = JSON.stringify({ notes, zIndexCounter }, null, 2);
      const blob = new Blob([dataStr], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url; a.download = 'sticky-notes.json'; a.click();
      URL.revokeObjectURL(url);
    }

    function importNotesFile(file) {
      const reader = new FileReader();
      reader.onload = () => {
        try {
          const parsed = JSON.parse(reader.result);
          if (Array.isArray(parsed.notes)) {
            parsed.notes.forEach(n => { n.id = n.id || uid(); notes.push(n); });
            zIndexCounter = Math.max(zIndexCounter, parsed.zIndexCounter || 1);
            renderAll(); save();
            alert('Imported ' + parsed.notes.length + ' notes');
          } else alert('Invalid file format');
        } catch (e) { alert('Import failed: ' + e.message) }
      };
      reader.readAsText(file);
    }

    newNoteBtn.addEventListener('click', () => createNote());
    clearAllBtn.addEventListener('click', clearBoard);
    exportBtn.addEventListener('click', exportNotes);
    importBtn.addEventListener('click', () => importFile.click());
    importFile.addEventListener('change', (ev) => { const f = ev.target.files[0]; if (f) importNotesFile(f); importFile.value = ''; });
    searchInput.addEventListener('input', () => {
      const q = searchInput.value.trim().toLowerCase();
      notes.forEach(n => {
        const el = document.querySelector(`[data-id="${n.id}"]`);
        if (!el) return;
        el.style.display = !q || (n.content || '').toLowerCase().includes(q) ? '' : 'none';
      });
    });

    fontSizeSelect.addEventListener('change', () => {
      const size = parseInt(fontSizeSelect.value);
      notes.forEach(n => { n.fontSize = size; updateNoteElement(n); });
      save();
    });

    load(); renderAll();
    if (notes.length === 0) {
      createNote({ content: 'Welcome! Drag the header to move. Resize with the corner. Change color & font size with controls.' });
    }

    window.addEventListener('beforeunload', save);
  </script>
</body>

</html>
