"use strict";
/* ===== PASTE YOUR SUPABASE VALUES HERE (public anon/publishable key only) ===== */
const SUPABASE_URL = "https://gtgvasosegnsarowaxoo.supabase.co";
const SUPABASE_ANON_KEY = "sb_publishable_XNxLiP1sdDY8Cjxnrej9HA_u3bt8CaC";
/* ================================================================================ */

const $ = (s, r = document) => r.querySelector(s), $$ = (s, r = document) => [...r.querySelectorAll(s)];
const esc = s => String(s ?? "").replace(/[&<>"']/g, c => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[c]));
const LANGS = ["python", "javascript", "html", "css", "sql", "json", "cpp", "java", "text"];
const EXT = { python: "py", javascript: "js", html: "html", css: "css", sql: "sql", json: "json", cpp: "cpp", java: "java", text: "txt" };
const TYPES = { link: ["Links", "link"], code: ["Code", "code-2"], question: ["Questions", "help-circle"], note: ["Notes", "file-text"] };
const TITLES = { home: "Home", all: "All Items", link: "Links", code: "Code", question: "Questions", note: "Notes", folders: "Folders", favs: "Favourites", bin: "Recycle Bin", settings: "Settings" };
const S = { user: null, items: [], folders: [], view: "home", q: "", folder: "", tag: "", sort: "upd", type: "", fav: false, loading: true, err: "" };
let sb = null, E = null, busy = false, qt = null;

const live = () => S.items.filter(i => !i.deleted_at), binned = () => S.items.filter(i => i.deleted_at);
const daysLeft = i => Math.max(0, 30 - Math.floor((Date.now() - new Date(i.deleted_at)) / 864e5));
const show = id => ["setup", "auth", "app"].forEach(x => $("#" + x).classList.toggle("hidden", x !== id));
const sleep = ms => new Promise(r => setTimeout(r, ms));
const isHttp = u => /^https?:\/\//i.test(u || "");
function toast(m, err) { const t = document.createElement("div"); t.className = "toast" + (err ? " err" : ""); t.textContent = m; $("#toasts").append(t); setTimeout(() => t.remove(), err ? 5000 : 2500); }
async function copy(t) { try { await navigator.clipboard.writeText(t || ""); toast("Copied"); } catch { toast("Clipboard blocked by the browser", true); } }
function download(name, text, type) { const a = document.createElement("a"); a.href = URL.createObjectURL(new Blob([text], { type })); a.download = name; document.body.append(a); a.click(); a.remove(); setTimeout(() => URL.revokeObjectURL(a.href), 1000); }

function dialog({ title, text = "", input, select, ok = "OK", danger }) {
  return new Promise(res => {
    const o = $("#confirm");
    o.innerHTML = `<div class="dlg glass" role="dialog" aria-modal="true"><h3 style="color:#fff;font-size:18px">${esc(title)}</h3><p class="muted">${esc(text)}</p>${input !== undefined ? `<input id="dv" value="${esc(input)}" maxlength="80">` : ""}${select ? `<select id="dv">${select.map(([v, l]) => `<option value="${esc(v)}">${esc(l)}</option>`).join("")}</select>` : ""}<div class="row end"><button class="btn" data-r="0">Cancel</button><button class="btn ${danger ? "danger" : "primary"}" data-r="1">${esc(ok)}</button></div></div>`;
    o.classList.remove("hidden");
    const dv = $("#dv", o); if (dv) dv.focus();
    o.onclick = e => { const b = e.target.closest("[data-r]"); if (!b && e.target !== o) return; o.classList.add("hidden"); o.innerHTML = ""; res(b && b.dataset.r === "1" ? (dv ? dv.value : true) : null); };
  });
}

/* ---------- data ---------- */
async function fetchAll(t) {
  const out = []; let f = 0;
  for (;;) { const { data, error } = await sb.from(t).select("*").order("created_at").range(f, f + 999); if (error) throw error; out.push(...data); if (data.length < 1000) return out; f += 1000; }
}
async function load() {
  S.err = "";
  try { const [a, b] = await Promise.all([fetchAll("vault_items"), fetchAll("vault_folders")]); S.items = a; S.folders = b.sort((x, y) => x.name.localeCompare(y.name)); }
  catch (e) { S.err = (e.message || "Network error") + " (if a table is missing, run the setup SQL)."; }
  S.loading = false; render();
}
async function upd(id, patch, msg) {
  const { data, error } = await sb.from("vault_items").update(patch).eq("id", id).select().single();
  if (error) { toast(error.message, true); return false; }
  S.items = S.items.map(x => x.id === id ? data : x); render(); if (msg) toast(msg); return true;
}

/* ---------- filtering & views ---------- */
function filtered() {
  let r = live(); const v = S.view;
  if (TYPES[v]) r = r.filter(i => i.type === v);
  if (v === "favs" || S.fav) r = r.filter(i => i.favourite);
  if (S.type) r = r.filter(i => i.type === S.type);
  if (S.folder === "none") r = r.filter(i => !i.folder_id); else if (S.folder) r = r.filter(i => i.folder_id === S.folder);
  if (S.tag) r = r.filter(i => (i.tags || []).includes(S.tag));
  const q = S.q.trim().toLowerCase();
  if (q) r = r.filter(i => [i.title, i.url, i.description, i.content, (i.tags || []).join(" ")].some(x => (x || "").toLowerCase().includes(q)));
  const o = { new: (a, b) => b.created_at.localeCompare(a.created_at), upd: (a, b) => b.updated_at.localeCompare(a.updated_at), az: (a, b) => a.title.localeCompare(b.title) }[S.sort];
  return r.sort(o);
}
const empty = (t, p, btn = true) => `<div class="empty glass"><h3 style="color:#fff">${t}</h3><p class="muted">${p}</p>${btn ? '<button class="btn primary" data-act="new">Add an item</button>' : ""}</div>`;
function card(i, bin) {
  const f = S.folders.find(x => x.id === i.folder_id);
  const sub = i.type === "link" ? i.url : i.type === "question" ? (i.content || "") : (i.description || i.content || "");
  const acts = bin ? [["restore", "rotate-ccw", "Restore"], ["purge", "trash-2", "Delete forever"]]
    : [["fav", "star", i.favourite ? "Unfavourite" : "Favourite"], ["copy", "copy", "Copy"], ["edit", "pencil", "Edit"], ["dup", "files", "Duplicate"], ["move", "folder-input", "Move to folder"], ["del", "trash-2", "Delete"]];
  return `<article class="card glass" data-id="${i.id}"><div class="ct" data-act="open"><i data-lucide="${TYPES[i.type][1]}"></i><div><h4>${esc(i.title)}</h4><p>${esc((sub || "").slice(0, 160))}</p></div></div><div class="meta">${f ? `<span class="chip">${esc(f.name)}</span>` : ""}${i.type === "code" ? `<span class="chip">${esc(i.language)}</span>` : ""}${(i.tags || []).map(t => `<span class="chip">#${esc(t)}</span>`).join("")}${bin ? `<span class="chip">${daysLeft(i)} days left</span>` : ""}</div><div class="acts">${acts.map(([a, ic, l]) => `<button class="ib${a === "fav" && i.favourite ? " on" : ""}" data-act="${a}" title="${l}" aria-label="${l}"><i data-lucide="${ic}"></i></button>`).join("")}</div></article>`;
}
const stat = (l, n) => `<div class="glass stat"><b>${n}</b><span>${l}</span></div>`;
const section = (t, a) => `<h3>${t}</h3>` + (a.length ? `<div class="grid">${a.map(i => card(i)).join("")}</div>` : '<p class="muted">Nothing here yet.</p>');
function home() {
  const L = live(), c = t => L.filter(i => i.type === t).length;
  if (!L.length && !S.folders.length && !binned().length) return empty("Your vault is empty", "Save your first link, snippet, question or note.");
  const fs = S.folders.map(f => `<button class="chip big" data-act="openFolder" data-fid="${f.id}">${esc(f.name)} · ${L.filter(i => i.folder_id === f.id).length}</button>`).join("");
  return `<div class="stats">${stat("Total items", L.length)}${stat("Links", c("link"))}${stat("Code snippets", c("code"))}${stat("Notes & questions", c("note") + c("question"))}${stat("In Recycle Bin", binned().length)}</div>${section("Pinned", L.filter(i => i.favourite).slice(0, 6))}${section("Recently added", [...L].sort((a, b) => b.created_at.localeCompare(a.created_at)).slice(0, 6))}<h3>Folders</h3><div class="row wrap">${fs || '<p class="muted">No folders yet.</p>'}</div>`;
}
function listView() {
  const tags = [...new Set(live().flatMap(i => i.tags || []))].sort();
  const sel = (k, o) => `<select data-f="${k}" aria-label="${k}">${o.map(([v, l]) => `<option value="${esc(v)}"${S[k] === v ? " selected" : ""}>${esc(l)}</option>`).join("")}</select>`;
  const tb = `<div class="tools">${["all", "favs"].includes(S.view) ? sel("type", [["", "All types"], ...Object.entries(TYPES).map(([k, v]) => [k, v[0]])]) : ""}${sel("folder", [["", "All folders"], ["none", "Unfiled"], ...S.folders.map(f => [f.id, f.name])])}${sel("tag", [["", "All tags"], ...tags.map(t => [t, "#" + t])])}${sel("sort", [["upd", "Recently updated"], ["new", "Newest"], ["az", "A–Z"]])}${S.view !== "favs" ? `<label class="chk"><input type="checkbox" data-f="fav"${S.fav ? " checked" : ""}> Favourites</label>` : ""}<button class="btn" data-act="clear">Clear filters</button></div>`;
  const r = filtered(), flt = S.q || S.folder || S.tag || S.fav || S.type;
  return tb + (r.length ? `<div class="grid">${r.map(i => card(i)).join("")}</div>` : empty(flt ? "No matches" : "Nothing here yet", flt ? "Try clearing the search or filters." : "Add something to get started.", !flt));
}
function foldersView() {
  const L = live();
  const row = (name, n, id) => `<article class="card glass"><div class="ct" data-act="openFolder" data-fid="${id}"><i data-lucide="folder"></i><div><h4>${esc(name)}</h4><p>${n} item${n === 1 ? "" : "s"}</p></div></div>${id === "none" ? "" : `<div class="acts"><button class="ib" data-act="renameFolder" data-fid="${id}" title="Rename" aria-label="Rename"><i data-lucide="pencil"></i></button><button class="ib" data-act="delFolder" data-fid="${id}" title="Delete" aria-label="Delete"><i data-lucide="trash-2"></i></button></div>`}</article>`;
  return `<div class="row"><button class="btn primary" data-act="newFolder"><i data-lucide="folder-plus"></i> New folder</button></div><div class="grid">${row("Unfiled", L.filter(i => !i.folder_id).length, "none")}${S.folders.map(f => row(f.name, L.filter(i => i.folder_id === f.id).length, f.id)).join("")}</div>`;
}
function binView() {
  const b = binned().sort((x, y) => y.deleted_at.localeCompare(x.deleted_at));
  return `<p class="muted">Deleted items stay here for 30 days, then a scheduled server job removes them permanently.</p>` + (b.length ? `<div class="row"><button class="btn danger" data-act="emptyBin">Empty Recycle Bin</button></div><div class="grid">${b.map(i => card(i, true)).join("")}</div>` : empty("Recycle Bin is empty", "Deleted items will appear here.", false));
}
function settings() {
  return `<div class="glass panel"><h3>Account</h3><p>${esc(S.user.email)}</p><div class="row"><button class="btn" data-act="signout">Sign out</button></div></div><div class="glass panel" style="max-width:none"><h3>Backup</h3><p class="muted">Backups include your data only, never credentials or tokens. Import never overwrites existing data.</p><div class="row wrap"><button class="btn primary" data-act="export" data-all="1">Export full backup (with Recycle Bin)</button><button class="btn" data-act="export">Export active items only</button><button class="btn" data-act="import">Import backup…</button><input type="file" id="imp" accept="application/json,.json" hidden></div></div>`;
}
function render() {
  $$("#nav [data-view]").forEach(b => b.classList.toggle("on", b.dataset.view === S.view));
  let h = `<h2>${TITLES[S.view]}</h2>`;
  if (S.loading) h += '<p class="muted">Loading your vault…</p>';
  else if (S.err) h += `<div class="empty glass"><h3 style="color:#fff">Couldn't load your vault</h3><p class="muted">${esc(S.err)}</p><button class="btn primary" data-act="retry">Retry</button></div>`;
  else h += { home, folders: foldersView, bin: binView, settings }[S.view]?.() ?? listView();
  $("#view").innerHTML = h; lucide.createIcons();
  if (!E && !$("#modal").classList.contains("hidden")) { const id = $("#modal .sheet")?.dataset.id; const it = S.items.find(x => x.id === id); it ? openView(id) : closeModal(); }
}

/* ---------- view modal & preview ---------- */
function openView(id) {
  const i = S.items.find(x => x.id === id); if (!i) return;
  const f = S.folders.find(x => x.id === i.folder_id); let b = "";
  const tags = (i.tags || []).map(t => `<span class="chip">#${esc(t)}</span>`).join("");
  if (i.type === "link") b = `<p>${isHttp(i.url) ? `<a href="${esc(i.url)}" target="_blank" rel="noopener noreferrer">${esc(i.url)}</a>` : esc(i.url)}</p><p class="pre">${esc(i.description)}</p><div class="row wrap"><button class="btn" data-act="copyContent"><i data-lucide="copy"></i> Copy URL</button></div>`;
  if (i.type === "code") {
    const l = i.language, pv = l === "html" || l === "css" ? '<button class="btn" data-act="preview">Preview</button>' : l === "javascript" ? '<button class="btn" data-act="run">Run (sandboxed)</button>' : l === "json" ? '<button class="btn" data-act="fmt">Format &amp; validate</button>' : l === "python" ? '<span class="muted">Python can\'t run safely in the browser here: highlighting only.</span>' : "";
    b = `<p class="pre">${esc(i.description)}</p><pre class="code"><code id="vc" class="language-${l === "text" ? "none" : l}">${esc(i.content)}</code></pre><div class="row wrap"><button class="btn" data-act="copyContent"><i data-lucide="copy"></i> Copy code</button><button class="btn" data-act="download"><i data-lucide="download"></i> Download</button>${pv}</div><div id="pv"></div>`;
  }
  if (i.type === "question") b = `<h3>Answer</h3><div class="pre">${esc(i.content)}</div><div class="row wrap"><button class="btn" data-act="copyTitle">Copy question</button><button class="btn" data-act="copyContent">Copy answer</button></div>`;
  if (i.type === "note") b = `<div class="pre">${esc(i.content)}</div><div class="row"><button class="btn" data-act="copyContent">Copy note</button></div>`;
  const acts = i.deleted_at ? `<p class="muted">In the Recycle Bin. ${daysLeft(i)} days left.</p>` : `<div class="acts"><button class="btn" data-act="edit"><i data-lucide="pencil"></i> Edit</button><button class="btn" data-act="fav"><i data-lucide="star"></i> ${i.favourite ? "Unfavourite" : "Favourite"}</button><button class="btn" data-act="dup">Duplicate</button><button class="btn" data-act="move">Move</button><button class="btn" data-act="del">Delete</button></div>`;
  $("#modal").innerHTML = `<div class="sheet glass" data-id="${i.id}"><div class="row between"><h3 style="color:#fff;font-size:19px">${esc(i.title)}</h3><button class="ib" data-act="closeM" aria-label="Close"><i data-lucide="x"></i></button></div><div class="meta">${f ? `<span class="chip">${esc(f.name)}</span>` : ""}${i.type === "code" ? `<span class="chip">${esc(i.language)}</span>` : ""}${tags}</div>${b}${acts}</div>`;
  $("#modal").classList.remove("hidden"); lucide.createIcons();
  const c = $("#vc"); if (c && window.Prism && i.language !== "text") try { Prism.highlightElement(c); } catch { }
}
function frame(doc) { const p = $("#pv"); p.innerHTML = '<iframe sandbox="allow-scripts" title="Sandboxed preview" referrerpolicy="no-referrer"></iframe>'; $("iframe", p).srcdoc = doc; }
const jsDoc = c => `<body style="font:13px monospace;margin:8px"><pre id="o"></pre><script>const o=document.getElementById("o"),w=(...a)=>{o.textContent+=a.map(x=>typeof x==="object"?JSON.stringify(x):String(x)).join(" ")+"\\n"};console.log=console.info=console.warn=console.error=w;onerror=m=>w("Error: "+m);<\/script><script>${c.replace(/<\/script/gi, "<\\/script")}<\/script></body>`;

/* ---------- editor ---------- */
function setStatus(m, err) { const s = $("#es"); if (s) { s.textContent = m; s.classList.toggle("errtxt", !!err); } }
function openEditor(item, type) {
  const t = item ? item.type : type, it = item || { tags: [], language: "javascript", folder_id: S.folder && S.folder !== "none" ? S.folder : null };
  E = { id: item ? item.id : null, type: t, dirty: false, saving: false, rev: 0, timer: null };
  const fi = (n, l, v, x = "") => `<label>${l}<input name="${n}" value="${esc(v || "")}" ${x}></label>`;
  const ta = (n, l, v, r, c = "") => `<label>${l}<textarea name="${n}" rows="${r}" class="${c}">${esc(v || "")}</textarea></label>`;
  let f = "";
  if (t === "link") f = fi("title", "Title", it.title, "required maxlength=300") + fi("url", "URL", it.url, 'type="url" required placeholder="https://…"') + ta("description", "Description", it.description, 3);
  if (t === "code") f = fi("title", "Title", it.title, "required maxlength=300") + `<label>Language<select name="language">${LANGS.map(l => `<option${l === it.language ? " selected" : ""}>${l}</option>`).join("")}</select></label>` + ta("content", "Code", it.content, 12, "mono") + ta("description", "Description", it.description, 2);
  if (t === "question") f = fi("title", "Question", it.title, "required maxlength=300") + ta("content", "Answer / explanation", it.content, 9);
  if (t === "note") f = fi("title", "Title", it.title, "required maxlength=300") + ta("content", "Note (plain text or Markdown)", it.content, 14);
  f += `<label>Folder<select name="folder_id"><option value="">Unfiled</option>${S.folders.map(x => `<option value="${x.id}"${x.id === it.folder_id ? " selected" : ""}>${esc(x.name)}</option>`).join("")}</select></label>` + fi("tags", "Tags (comma separated)", (it.tags || []).join(", ")) + `<label class="chk"><input type="checkbox" name="favourite"${it.favourite ? " checked" : ""}> Favourite</label>`;
  $("#modal").innerHTML = `<div class="sheet glass" data-id="${E.id || ""}"><div class="row between"><h3 style="color:#fff;font-size:19px">${item ? "Edit" : "New"} ${TYPES[t][0].replace(/s$/, "")}</h3><button class="ib" data-act="closeM" aria-label="Close"><i data-lucide="x"></i></button></div><form id="ef">${f}<div class="row between"><span id="es" class="muted" role="status">${t === "note" ? "Autosave on" : ""}</span><span class="row"><button type="button" class="btn" data-act="closeM">Cancel</button><button class="btn primary" type="submit">Save</button></span></div></form></div>`;
  $("#modal").classList.remove("hidden"); lucide.createIcons();
}
function readForm() {
  const f = new FormData($("#ef")), g = k => (f.get(k) || "").toString(), fail = m => { setStatus(m, true); return null; };
  const title = g("title").trim(); if (!title) return fail("Title is required");
  const rec = { type: E.type, title, folder_id: g("folder_id") || null, favourite: f.has("favourite"), tags: [...new Set(g("tags").split(",").map(s => s.trim().replace(/^#/, "").toLowerCase()).filter(Boolean))].slice(0, 20) };
  if (E.type === "link") { const u = g("url").trim(); if (!isHttp(u)) return fail("Enter a valid http(s) URL"); try { new URL(u); } catch { return fail("Enter a valid URL"); } rec.url = u; rec.description = g("description"); }
  else {
    rec.content = g("content");
    if (E.type === "code") { rec.language = g("language"); rec.description = g("description"); if (!rec.content.trim()) return fail("Code is required"); if (rec.language === "json") try { JSON.parse(rec.content); } catch (e) { return fail("Invalid JSON: " + e.message); } }
    if (E.type === "question" && !rec.content.trim()) return fail("Answer is required");
  }
  return rec;
}
async function saveItem(silent) {
  while (E && E.saving) await sleep(100);
  if (!E) return false;
  const rec = readForm(); if (!rec) return false;
  const rev = E.rev; E.saving = true; setStatus("Saving…");
  const q = E.id ? sb.from("vault_items").update(rec).eq("id", E.id) : sb.from("vault_items").insert({ ...rec, owner_id: S.user.id });
  const r = await q.select().single();
  if (!E) return false;
  E.saving = false;
  if (r.error) { setStatus("Not saved: " + r.error.message + " (your edits are kept)", true); if (!silent) toast("Save failed", true); return false; }
  E.id = r.data.id; $("#modal .sheet").dataset.id = E.id;
  const ix = S.items.findIndex(x => x.id === E.id); ix < 0 ? S.items.push(r.data) : (S.items[ix] = r.data);
  if (E.rev === rev) { E.dirty = false; setStatus("Saved ✓ " + new Date().toLocaleTimeString()); } else setStatus("Unsaved changes…");
  render(); if (!silent) toast("Saved");
  if (!silent && E.type !== "note" && !E.dirty) closeModal();
  return true;
}
function closeModal() { if (E) clearTimeout(E.timer); E = null; const m = $("#modal"); m.classList.add("hidden"); m.innerHTML = ""; }

/* ---------- actions ---------- */
const A = {
  nav(el) { S.view = el.dataset.view; S.folder = S.tag = S.type = ""; S.fav = false; render(); },
  retry() { S.loading = true; render(); return load(); },
  async new() { const t = TYPES[S.view] ? S.view : await dialog({ title: "New item", text: "What do you want to save?", select: Object.entries(TYPES).map(([k, v]) => [k, v[0]]), ok: "Continue" }); if (t) openEditor(null, t); },
  open(el, it) { if (it.type === "link" && !it.deleted_at && isHttp(it.url)) window.open(it.url, "_blank", "noopener,noreferrer"); else openView(it.id); },
  copy(el, it) { return copy(it.type === "link" ? it.url : it.type === "code" || it.type === "note" ? it.content : it.title); },
  copyContent(el, it) { return copy(it.type === "link" ? it.url : it.content); },
  copyTitle(el, it) { return copy(it.title); },
  edit(el, it) { closeModal(); openEditor(it); },
  async dup(el, it) {
    const { id, created_at, updated_at, deleted_at, ...r } = it;
    const { data, error } = await sb.from("vault_items").insert({ ...r, title: (it.title + " (copy)").slice(0, 300), favourite: false }).select().single();
    if (error) return toast(error.message, true); S.items.push(data); render(); toast("Duplicated");
  },
  fav(el, it) { return upd(it.id, { favourite: !it.favourite }, it.favourite ? "Removed from favourites" : "Added to favourites"); },
  async move(el, it) { const v = await dialog({ title: "Move to folder", select: [["", "Unfiled"], ...S.folders.map(f => [f.id, f.name])], ok: "Move" }); if (v !== null) await upd(it.id, { folder_id: v || null }, "Moved"); },
  async del(el, it) { if (await dialog({ title: "Delete this item?", text: `"${it.title}" moves to the Recycle Bin and is permanently deleted after 30 days. You can restore it until then.`, ok: "Move to Recycle Bin", danger: true })) await upd(it.id, { deleted_at: new Date().toISOString() }, "Moved to Recycle Bin"); },
  restore(el, it) { return upd(it.id, { deleted_at: null }, "Restored"); },
  async purge(el, it) {
    if (!await dialog({ title: "Delete forever?", text: `"${it.title}" will be permanently deleted. This cannot be undone.`, ok: "Delete forever", danger: true })) return;
    const { error } = await sb.from("vault_items").delete().eq("id", it.id); if (error) return toast(error.message, true);
    S.items = S.items.filter(x => x.id !== it.id); render(); toast("Permanently deleted");
  },
  async emptyBin() {
    const n = binned().length; if (!n || !await dialog({ title: "Empty Recycle Bin?", text: `${n} item(s) will be permanently deleted. This cannot be undone.`, ok: "Delete everything", danger: true })) return;
    const { error } = await sb.from("vault_items").delete().eq("owner_id", S.user.id).not("deleted_at", "is", null); if (error) return toast(error.message, true);
    S.items = live(); render(); toast("Recycle Bin emptied");
  },
  async closeM() { if (E && E.dirty && !await dialog({ title: "Discard unsaved changes?", text: "Your latest edits have not been saved.", ok: "Discard", danger: true })) return; closeModal(); },
  openFolder(el) { S.view = "all"; S.folder = el.dataset.fid; S.tag = S.type = ""; S.fav = false; render(); },
  async newFolder() {
    const n = await dialog({ title: "New folder", input: "", ok: "Create" }); if (n === null) return;
    const name = n.trim(); if (!name) return toast("Enter a folder name", true);
    const { data, error } = await sb.from("vault_folders").insert({ owner_id: S.user.id, name }).select().single();
    if (error) return toast(error.code === "23505" ? "A folder with that name already exists" : error.message, true);
    S.folders.push(data); S.folders.sort((a, b) => a.name.localeCompare(b.name)); render(); toast("Folder created");
  },
  async renameFolder(el) {
    const f = S.folders.find(x => x.id === el.dataset.fid), n = await dialog({ title: "Rename folder", input: f.name, ok: "Rename" }); if (n === null) return;
    if (!n.trim()) return toast("Enter a folder name", true);
    const { data, error } = await sb.from("vault_folders").update({ name: n.trim() }).eq("id", f.id).select().single();
    if (error) return toast(error.code === "23505" ? "A folder with that name already exists" : error.message, true);
    S.folders = S.folders.map(x => x.id === f.id ? data : x).sort((a, b) => a.name.localeCompare(b.name)); render(); toast("Folder renamed");
  },
  async delFolder(el) {
    const f = S.folders.find(x => x.id === el.dataset.fid), n = S.items.filter(i => i.folder_id === f.id).length;
    if (!await dialog({ title: "Delete folder?", text: `"${f.name}" will be deleted. Its ${n} item(s), including any in the Recycle Bin, move to Unfiled. No items are deleted.`, ok: "Delete folder", danger: true })) return;
    const { error } = await sb.from("vault_folders").delete().eq("id", f.id); if (error) return toast(error.message, true);
    S.folders = S.folders.filter(x => x.id !== f.id); S.items = S.items.map(i => i.folder_id === f.id ? { ...i, folder_id: null } : i); render(); toast("Folder deleted");
  },
  clear() { S.q = ""; $("#q").value = ""; S.folder = S.tag = S.type = ""; S.fav = false; render(); },
  preview(el, it) { frame(it.language === "css" ? `<!doctype html><style>${it.content.replace(/<\/style/gi, "")}</style><body><h1>Heading</h1><p>Paragraph with <a href="#">a link</a>.</p><button>Button</button><ul><li>One</li><li>Two</li></ul></body>` : it.content); },
  run(el, it) { frame(jsDoc(it.content)); },
  fmt(el, it) { try { $("#pv").innerHTML = `<pre class="code">${esc(JSON.stringify(JSON.parse(it.content), null, 2))}</pre><p class="ok">Valid JSON</p>`; } catch (e) { $("#pv").innerHTML = `<p class="errtxt">Invalid JSON: ${esc(e.message)}</p>`; } },
  download(el, it) { download((it.title.replace(/[^\w.-]+/g, "_") || "snippet") + "." + EXT[it.language], it.content, "text/plain"); },
  async export(el) {
    const all = !!el.dataset.all, [items, folders] = await Promise.all([fetchAll("vault_items"), fetchAll("vault_folders")]), fn = new Map(folders.map(f => [f.id, f.name]));
    const out = { app: "platiee", schema_version: 1, exported_at: new Date().toISOString(), scope: all ? "full" : "active", folders: folders.map(f => ({ name: f.name })), items: items.filter(i => all || !i.deleted_at).map(i => ({ type: i.type, title: i.title, url: i.url, language: i.language, content: i.content, description: i.description, tags: i.tags, favourite: i.favourite, created_at: i.created_at, updated_at: i.updated_at, deleted_at: i.deleted_at, folder: fn.get(i.folder_id) || null })) };
    download(`platiee-backup-${new Date().toISOString().slice(0, 10)}.json`, JSON.stringify(out, null, 2), "application/json"); toast("Backup downloaded");
  },
  import() { $("#imp").click(); },
  async signout() { const { error } = await sb.auth.signOut(); if (error) toast(error.message, true); }
};

async function importFile(file) {
  let d; try { d = JSON.parse(await file.text()); } catch { return toast("That file is not valid JSON", true); }
  if (!d || d.app !== "platiee" || d.schema_version !== 1 || !Array.isArray(d.items)) return toast("Not a Platiee v1 backup", true);
  const str = x => x == null ? null : String(x);
  const ok = d.items.filter(i => i && TYPES[i.type] && typeof i.title === "string" && i.title.trim() && i.title.length <= 300 && (i.type !== "link" || isHttp(i.url)) && (i.type !== "code" || (LANGS.includes(i.language) && typeof i.content === "string")));
  const key = i => [i.type, i.title, i.url || i.content || ""].join("|"), have = new Set(S.items.map(key)), dup = ok.filter(i => have.has(key(i)));
  const mode = await dialog({ title: "Import preview", text: `${d.items.length} in file · ${d.items.length - ok.length} invalid (will be skipped) · ${dup.length} duplicates · ${ok.length - dup.length} new. Existing data is never overwritten.`, select: [["skip", "Skip duplicates"], ["copy", "Import duplicates as copies"]], ok: "Import" });
  if (!mode) return;
  const names = [...new Set(ok.map(i => i.folder).filter(Boolean).map(String))], fmap = Object.fromEntries(S.folders.map(f => [f.name, f.id])), miss = names.filter(n => !(n in fmap) && n.length <= 80);
  if (miss.length) { const { data, error } = await sb.from("vault_folders").insert(miss.map(name => ({ owner_id: S.user.id, name }))).select(); if (error) return toast("Folder import failed: " + error.message, true); data.forEach(f => { fmap[f.name] = f.id; S.folders.push(f); }); }
  const rows = (mode === "skip" ? ok.filter(i => !have.has(key(i))) : ok).map(i => ({ owner_id: S.user.id, type: i.type, title: i.title, url: i.type === "link" ? i.url : null, language: i.type === "code" ? i.language : null, content: str(i.content), description: str(i.description), tags: Array.isArray(i.tags) ? i.tags.map(String).slice(0, 20) : [], favourite: !!i.favourite, deleted_at: i.deleted_at && !isNaN(Date.parse(i.deleted_at)) ? i.deleted_at : null, folder_id: fmap[i.folder] || null }));
  let done = 0, failed = 0;
  for (let n = 0; n < rows.length; n += 100) { const { data, error } = await sb.from("vault_items").insert(rows.slice(n, n + 100)).select(); if (error) failed += rows.slice(n, n + 100).length; else { done += data.length; S.items.push(...data); } }
  S.folders.sort((a, b) => a.name.localeCompare(b.name)); render();
  toast(`Imported ${done}, skipped ${d.items.length - rows.length}, failed ${failed}`, failed > 0);
}

/* ---------- events (registered once) ---------- */
document.addEventListener("click", async e => {
  const el = e.target.closest("[data-act],[data-view]");
  if (!el) { if (e.target.id === "modal") A.closeM(); return; }
  if (busy) return;
  const act = el.dataset.act || "nav", host = el.closest("[data-id]"), it = host && S.items.find(x => x.id === host.dataset.id);
  if (!A[act]) return;
  busy = true; document.body.classList.add("busy");
  try { await A[act](el, it); } catch (err) { toast(err.message || "Something went wrong", true); } finally { busy = false; document.body.classList.remove("busy"); }
});
document.addEventListener("change", e => {
  const t = e.target;
  if (t.id === "imp" && t.files[0]) { importFile(t.files[0]).catch(err => toast(err.message, true)); t.value = ""; return; }
  if (t.dataset.f) { S[t.dataset.f] = t.type === "checkbox" ? t.checked : t.value; render(); }
});
document.addEventListener("input", e => {
  if (!E || !e.target.closest("#ef")) return;
  E.dirty = true; E.rev++; setStatus("Unsaved changes…");
  if (E.type === "note") { clearTimeout(E.timer); E.timer = setTimeout(() => saveItem(true), 1500); }
});
document.addEventListener("submit", async e => {
  e.preventDefault();
  if (e.target.id === "ef") { if (busy) return; busy = true; try { await saveItem(false); } finally { busy = false; } }
  if (e.target.id === "af") signIn(e);
});
document.addEventListener("keydown", e => { if (e.key !== "Escape") return; if (!$("#confirm").classList.contains("hidden")) $("#confirm").click(); else if (!$("#modal").classList.contains("hidden")) A.closeM(); });
window.addEventListener("beforeunload", e => { if (E && E.dirty) { e.preventDefault(); e.returnValue = ""; } });
$("#q").addEventListener("input", e => { clearTimeout(qt); qt = setTimeout(() => { S.q = e.target.value; if (S.q.trim() && !TYPES[S.view] && !["all", "favs"].includes(S.view)) S.view = "all"; render(); }, 200); });

/* ---------- auth & startup ---------- */
async function signIn(e) {
  const m = $("#am"), mode = e.submitter ? e.submitter.value : "signin", email = $("#em").value.trim(), password = $("#pw").value, bs = $$("#af button");
  if (!email || password.length < 8) { m.textContent = "Enter an email and a password of at least 8 characters."; return; }
  m.textContent = "Working…"; bs.forEach(b => b.disabled = true);
  const { data, error } = mode === "signup" ? await sb.auth.signUp({ email, password, options: { emailRedirectTo: location.href.split("#")[0] } }) : await sb.auth.signInWithPassword({ email, password });
  bs.forEach(b => b.disabled = false);
  m.textContent = error ? error.message : mode === "signup" && !data.session ? "Account created. Check your email to confirm, then sign in." : "";
}
function gate(session) {
  if (!session) { S.user = null; S.items = []; S.folders = []; closeModal(); show("auth"); return; }
  const first = !S.user || S.user.id !== session.user.id; S.user = session.user; show("app");
  if (first) { S.loading = true; render(); load(); }
}
function init() {
  lucide.createIcons();
  if (!SUPABASE_URL || !SUPABASE_ANON_KEY || !window.supabase) return show("setup");
  try { sb = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY); } catch { return show("setup"); }
  if (window.Prism && Prism.plugins.autoloader) Prism.plugins.autoloader.languages_path = "https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/";
  sb.auth.onAuthStateChange((ev, s) => setTimeout(() => gate(s), 0));
}
init();
