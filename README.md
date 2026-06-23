<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Task Board — Otternative Marketing</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🦦</text></svg>">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@latest/tabler-icons.min.css" />
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    background: #e8e8e4;
    background-image:
      repeating-linear-gradient(0deg, transparent, transparent 24px, rgba(0,0,0,0.02) 24px, rgba(0,0,0,0.02) 25px),
      repeating-linear-gradient(90deg, transparent, transparent 24px, rgba(0,0,0,0.015) 24px, rgba(0,0,0,0.015) 25px);
    min-height: 100vh; padding: 32px 16px 60px;
  }
  .app { max-width: 1100px; margin: 0 auto; }

  /* ── Header ── */
  .header {
    background: linear-gradient(180deg, #2a2a2a 0%, #1a1a1a 50%, #111 100%);
    border-radius: 14px 14px 0 0; padding: 20px 26px;
    display: flex; align-items: center; justify-content: space-between;
    box-shadow: 0 4px 0 #000, 0 6px 24px rgba(0,0,0,0.4), inset 0 1px 0 rgba(255,255,255,0.1);
  }
  .brand { display: flex; align-items: center; gap: 12px; }
  .brand-icon {
    width: 34px; height: 34px; border-radius: 8px;
    background: linear-gradient(145deg, #fff 0%, #e0e0e0 100%);
    box-shadow: inset 0 1px 0 rgba(255,255,255,0.9), 0 2px 6px rgba(0,0,0,0.4);
    display: flex; align-items: center; justify-content: center; color: #1a1a1a; font-size: 17px;
  }
  .brand-title { font-size: 16px; font-weight: 700; color: #fff; letter-spacing: 0.02em; }
  .brand-sub { font-size: 11px; color: #888; margin-top: 1px; }
  .header-right { display: flex; align-items: center; gap: 10px; }
  .user-badge {
    font-size: 11px; color: #888; background: rgba(255,255,255,0.07);
    border: 1px solid rgba(255,255,255,0.1); border-radius: 20px;
    padding: 4px 10px; display: flex; align-items: center; gap: 5px;
  }
  .btn-add {
    display: flex; align-items: center; gap: 6px; padding: 8px 17px;
    background: linear-gradient(180deg, #fff 0%, #e8e8e4 100%);
    border: 1px solid #bbb; border-radius: 8px;
    color: #1a1a1a; font-size: 13px; font-weight: 600; font-family: inherit; cursor: pointer;
    box-shadow: 0 3px 0 #aaa, 0 4px 10px rgba(0,0,0,0.2), inset 0 1px 0 rgba(255,255,255,0.9);
    transition: all 0.1s;
  }
  .btn-add:hover { background: linear-gradient(180deg, #fff 0%, #f0f0ec 100%); }
  .btn-add:active { transform: translateY(2px); box-shadow: 0 1px 0 #aaa; }

  /* ── Body panel ── */
  .body-panel {
    background: linear-gradient(180deg, #ffffff 0%, #f5f5f2 100%);
    border: 1px solid #d0d0cc; border-top: none; border-radius: 0 0 14px 14px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.18); padding: 24px 24px 28px;
  }

  /* ── Stats ── */
  .stat-bar { display: grid; grid-template-columns: repeat(4,1fr); gap: 12px; margin-bottom: 24px; }
  .stat {
    background: linear-gradient(180deg, #fafaf8 0%, #efefeb 100%);
    border: 1px solid #d8d8d4; border-radius: 10px; padding: 14px 16px; text-align: center;
    box-shadow: 0 2px 5px rgba(0,0,0,0.08), inset 0 1px 0 rgba(255,255,255,1);
  }
  .stat-label { font-size: 10px; color: #999; text-transform: uppercase; letter-spacing: 0.08em; margin-bottom: 5px; }
  .stat-value { font-size: 26px; font-weight: 700; color: #1a1a1a; }

  /* ── Search ── */
  .search-wrap { position: relative; margin-bottom: 14px; }
  .search-wrap .ti { position: absolute; left: 10px; top: 50%; transform: translateY(-50%); color: #aaa; font-size: 15px; pointer-events: none; }
  .search-input {
    width: 100%; padding: 8px 11px 8px 32px;
    border: 1px solid #d0d0cc; border-radius: 8px;
    font-size: 13px; font-family: inherit; color: #1a1a1a;
    background: linear-gradient(180deg, #fafaf8 0%, #f5f5f2 100%);
    box-shadow: inset 0 2px 5px rgba(0,0,0,0.06);
  }
  .search-input:focus { outline: none; border-color: #888; box-shadow: inset 0 2px 5px rgba(0,0,0,0.06), 0 0 0 3px rgba(0,0,0,0.07); }
  .search-input::placeholder { color: #bbb; }

  /* ── Filter rows ── */
  .filter-rows { display: flex; flex-direction: column; gap: 8px; margin-bottom: 14px; }
  .filter-row { display: flex; align-items: center; gap: 8px; }
  .filter-row-label {
    font-size: 10px; font-weight: 700; color: #aaa; text-transform: uppercase;
    letter-spacing: 0.08em; width: 62px; flex-shrink: 0;
  }
  .filter-pills { display: flex; gap: 5px; flex-wrap: wrap; }
  .fil-btn {
    padding: 5px 12px; border-radius: 20px; border: 1px solid #d0d0cc;
    background: linear-gradient(180deg, #fafaf8 0%, #e8e8e4 100%);
    font-size: 12px; font-weight: 500; font-family: inherit; cursor: pointer; color: #666;
    box-shadow: 0 2px 4px rgba(0,0,0,0.08), inset 0 1px 0 rgba(255,255,255,0.9);
    transition: all 0.12s; white-space: nowrap;
  }
  .fil-btn:hover { color: #1a1a1a; border-color: #bbb; }
  .fil-btn.active {
    background: linear-gradient(180deg, #2a2a2a 0%, #1a1a1a 100%);
    border-color: #000; color: #fff;
    box-shadow: 0 1px 0 #000, inset 0 2px 4px rgba(0,0,0,0.35);
  }
  /* alias so category buttons share same style */
  .cat-btn { padding: 5px 12px; border-radius: 20px; border: 1px solid #d0d0cc;
    background: linear-gradient(180deg, #fafaf8 0%, #e8e8e4 100%);
    font-size: 12px; font-weight: 500; font-family: inherit; cursor: pointer; color: #666;
    box-shadow: 0 2px 4px rgba(0,0,0,0.08), inset 0 1px 0 rgba(255,255,255,0.9);
    transition: all 0.12s; white-space: nowrap;
  }
  .cat-btn:hover { color: #1a1a1a; border-color: #bbb; }
  .cat-btn.active {
    background: linear-gradient(180deg, #2a2a2a 0%, #1a1a1a 100%);
    border-color: #000; color: #fff;
    box-shadow: 0 1px 0 #000, inset 0 2px 4px rgba(0,0,0,0.35);
  }
  .count-chip { font-size: 10px; background: rgba(0,0,0,0.1); border-radius: 10px; padding: 1px 6px; margin-left: 3px; font-weight: 600; }
  .cat-btn.active .count-chip { background: rgba(255,255,255,0.15); }

  /* ── Table ── */
  .table-wrap { border: 1px solid #d0d0cc; border-radius: 10px; overflow: hidden; box-shadow: 0 3px 12px rgba(0,0,0,0.1); }
  table { width: 100%; border-collapse: collapse; }
  thead { background: linear-gradient(180deg, #2a2a2a 0%, #1a1a1a 100%); }
  th {
    padding: 10px 12px; font-size: 10px; font-weight: 700; color: #aaa;
    text-align: left; text-transform: uppercase; letter-spacing: 0.09em;
    border-bottom: 2px solid #000; white-space: nowrap;
  }
  td { padding: 11px 12px; font-size: 13px; color: #1a1a1a; border-bottom: 1px solid #efefeb; vertical-align: middle; }
  tr:nth-child(odd) td { background: rgba(255,255,255,0.7); }
  tr:nth-child(even) td { background: rgba(240,240,236,0.5); }
  tr:hover td { background: rgba(220,220,216,0.6) !important; }
  tr:last-child td { border-bottom: none; }
  tr.done td { opacity: 0.4; }
  tr.done .task-name { text-decoration: line-through; }

  /* ── Checkboxes ── */
  .cb {
    width: 16px; height: 16px; border-radius: 4px; border: 1px solid #bbb; cursor: pointer;
    appearance: none; -webkit-appearance: none;
    background: linear-gradient(180deg, #fff 0%, #e8e8e4 100%);
    box-shadow: inset 0 1px 3px rgba(0,0,0,0.15); vertical-align: middle; transition: all 0.15s;
  }
  .cb:checked {
    background: linear-gradient(180deg, #fff 0%, #efefeb 100%); border-color: #333;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12'%3E%3Cpath d='M2 6l3 3 5-5' stroke='%231a1a1a' stroke-width='2.2' fill='none' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E");
    background-size: 10px; background-repeat: no-repeat; background-position: center;
    box-shadow: inset 0 1px 3px rgba(0,0,0,0.1);
  }

  /* ── Task name & notes ── */
  .task-name { font-size: 13px; color: #1a1a1a; line-height: 1.5; word-break: break-word; }
  .task-notes { font-size: 11px; color: #aaa; font-style: italic; margin-top: 3px; line-height: 1.4; }

  /* ── Category badges ── */
  .cat-badge {
    display: inline-block; padding: 3px 9px; border-radius: 12px;
    font-size: 10px; font-weight: 700; letter-spacing: 0.04em; white-space: nowrap;
    box-shadow: inset 0 1px 0 rgba(255,255,255,0.5), 0 1px 3px rgba(0,0,0,0.12);
  }
  .badge-bm   { background: linear-gradient(180deg, #f0effe, #dddcf6); color: #3c3489; border: 1px solid #c8c4e8; }
  .badge-ad   { background: linear-gradient(180deg, #eef6fe, #cce0f8); color: #0c447c; border: 1px solid #a8ccee; }
  .badge-pp   { background: linear-gradient(180deg, #f0f8e8, #cce4a8); color: #27500a; border: 1px solid #a8d080; }
  .badge-perf { background: linear-gradient(180deg, #fef8e8, #f8dfa0); color: #633806; border: 1px solid #e8c870; }
  .badge-cw   { background: linear-gradient(180deg, #fef0f8, #f4c8dc); color: #72243e; border: 1px solid #e0a0be; }
  .badge-gen  { background: linear-gradient(180deg, #f5f5f2, #e0e0dc); color: #444441; border: 1px solid #c8c8c4; }

  /* ── Priority & Assignee selects ── */
  .pri-select {
    padding: 5px 8px; border-radius: 7px; font-size: 12px; font-family: inherit;
    font-weight: 600; cursor: pointer; min-width: 84px;
    box-shadow: inset 0 1px 3px rgba(0,0,0,0.12);
  }
  .pri-select.pri-high { background: linear-gradient(180deg, #fee8e8, #f5c1c1); border: 1px solid #e8a0a0; color: #a32d2d; }
  .pri-select.pri-med  { background: linear-gradient(180deg, #fef8e8, #fad880); border: 1px solid #e8c050; color: #633806; }
  .pri-select.pri-low  { background: linear-gradient(180deg, #eef6fe, #b8d8f4); border: 1px solid #90b8e0; color: #0c447c; }
  .assign-select {
    padding: 5px 8px; border-radius: 7px; border: 1px solid #d0d0cc;
    background: linear-gradient(180deg, #fafaf8 0%, #efefeb 100%);
    font-size: 12px; font-family: inherit; color: #1a1a1a; cursor: pointer; min-width: 105px;
  }

  /* ── Updated by ── */
  .updated-by { font-size: 11px; color: #bbb; white-space: nowrap; }

  /* ── Action buttons ── */
  .btn-del, .btn-edit {
    background: none; border: none; cursor: pointer; color: #ccc;
    padding: 5px; border-radius: 5px; line-height: 1; opacity: 0; font-size: 15px; transition: all 0.15s;
  }
  tr:hover .btn-del, tr:hover .btn-edit { opacity: 1; }
  .btn-del:hover { color: #a32d2d; background: linear-gradient(180deg, #fee8e8, #f5c1c1); }
  .btn-edit:hover { color: #1a1a1a; background: linear-gradient(180deg, #f0f0ec, #e0e0dc); }

  /* ── Loading / Empty ── */
  .loading, .empty { padding: 52px; text-align: center; color: #bbb; font-size: 13px; }

  /* ── Focus timer ── */
  .btn-focus {
    background: none; border: none; cursor: pointer; color: #ccc;
    padding: 5px; border-radius: 5px; line-height: 1; opacity: 0; font-size: 15px; transition: all 0.15s;
  }
  tr:hover .btn-focus { opacity: 1; }
  .btn-focus.running { color: #f59e0b; opacity: 1 !important; animation: focusPulse 1.5s infinite; }
  @keyframes focusPulse { 0%,100%{opacity:1} 50%{opacity:0.5} }
  .btn-focus:hover { color: #1a1a1a; background: linear-gradient(180deg, #f0f0ec, #e0e0dc); }
  .focus-timer-display {
    font-size: 48px; font-weight: 700; color: #fff; letter-spacing: 0.06em;
    font-variant-numeric: tabular-nums; text-align: center; margin: 18px 0 8px;
  }
  .focus-started-by { font-size: 11px; color: #666; text-align: center; margin-bottom: 20px; }
  .focus-sessions-wrap { margin-top: 12px; border-top: 1px dashed #e8e8e4; padding-top: 12px; }
  .focus-sessions-header { font-size: 11px; font-weight: 700; color: #888; margin-bottom: 7px; display: flex; align-items: center; gap: 5px; }
  .focus-session-row { display: flex; align-items: center; justify-content: space-between; padding: 5px 0; border-bottom: 1px solid #f2f2ef; font-size: 11px; gap: 8px; }
  .focus-session-row:last-of-type { border-bottom: none; }
  .fsw { color: #1a1a1a; font-weight: 600; min-width: 70px; }
  .fsd { color: #aaa; flex: 1; }
  .fsdur { color: #444; font-weight: 700; font-variant-numeric: tabular-nums; white-space: nowrap; }
  .focus-total-row { display: flex; justify-content: space-between; margin-top: 8px; padding-top: 8px; border-top: 2px solid #e0e0dc; font-size: 12px; font-weight: 700; color: #333; }

  /* ── Expand details ── */
  .detail-chips {
    display: inline-flex; align-items: center; gap: 5px; margin-top: 6px;
    cursor: pointer; flex-wrap: wrap;
  }
  .detail-chip {
    display: inline-flex; align-items: center; gap: 3px;
    padding: 3px 8px; border-radius: 20px; font-size: 10px; font-weight: 600;
    border: 1px solid; white-space: nowrap; letter-spacing: 0.01em;
  }
  .chip-note, .chip-voice, .chip-time { background: #f0fdf4; color: #16a34a; border-color: #bbf7d0; }
  .detail-chevron {
    font-size: 13px; color: #ccc; transition: transform 0.2s; display: inline-block; margin-left: 1px;
  }
  .detail-chips.open .detail-chevron { transform: rotate(180deg); }
  .detail-chips:hover .detail-chevron { color: #888; }
  .expand-row > td { padding: 0 !important; border-bottom: 1px solid #efefeb; }
  tr:nth-child(odd) + .expand-row > td { background: rgba(255,255,255,0.7); }
  tr:nth-child(even) + .expand-row > td { background: rgba(240,240,236,0.5); }
  .expand-content {
    padding: 12px 16px 14px 44px;
    border-top: 1px dashed #e0e0dc;
  }
  .expand-who { font-size: 11px; font-weight: 700; color: #888; margin-bottom: 7px; }
  .expand-who span { color: #1a1a1a; }
  .expand-notes-text { font-size: 12px; color: #555; line-height: 1.6; white-space: pre-wrap; margin-bottom: 4px; }
  .expand-audio { width: 100%; max-width: 320px; margin-top: 8px; accent-color: #1a1a1a; display: block; }

  /* ── Voice notes ── */
  .btn-voice {
    background: none; border: none; cursor: pointer; color: #ccc;
    padding: 5px; border-radius: 5px; line-height: 1; opacity: 0; font-size: 15px; transition: all 0.15s;
  }
  tr:hover .btn-voice { opacity: 1; }
  .btn-voice.has-voice { color: #22c55e; opacity: 1 !important; }
  .btn-voice:hover { color: #1a1a1a; background: linear-gradient(180deg, #f0f0ec, #e0e0dc); }
  .btn-voice.has-voice:hover { color: #16a34a; background: rgba(34,197,94,0.12); }
  .voice-overlay {
    display: none; position: fixed; inset: 0; z-index: 600;
    align-items: flex-end; justify-content: center; padding-bottom: 32px; pointer-events: none;
  }
  .voice-overlay.open { display: flex; pointer-events: all; }
  .voice-box {
    background: linear-gradient(180deg, #2a2a2a, #1a1a1a); border: 1px solid #000; border-radius: 14px;
    padding: 20px 24px; min-width: 320px; max-width: 90vw;
    box-shadow: 0 8px 32px rgba(0,0,0,0.4); animation: slideUp 0.2s ease;
  }
  .voice-task-name { font-size: 11px; color: #666; margin-bottom: 14px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .rec-indicator { display: flex; align-items: center; gap: 12px; margin-bottom: 18px; }
  .rec-dot { width: 11px; height: 11px; border-radius: 50%; background: #e74c3c; animation: blink 1s infinite; flex-shrink: 0; }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0.2} }
  .rec-timer { font-size: 30px; font-weight: 700; color: #fff; letter-spacing: 0.05em; font-variant-numeric: tabular-nums; }
  .voice-box-actions { display: flex; gap: 8px; justify-content: flex-end; }
  .vbtn { padding: 7px 16px; border-radius: 8px; font-size: 13px; font-family: inherit; cursor: pointer; }
  .vbtn-cancel { border: 1px solid #444; background: #333; color: #ccc; }
  .vbtn-stop { border: 1px solid #27ae60; background: linear-gradient(180deg,#2ecc71,#27ae60); color: #fff; font-weight: 600; box-shadow: 0 2px 0 #1e8449; }
  .vbtn-del { border: 1px solid #c0392b; background: linear-gradient(180deg,#e74c3c,#c0392b); color: #fff; font-weight: 600; }
  .vbtn-rerec { border: 1px solid #555; background: #3a3a3a; color: #ddd; }
  .vbtn-close { border: 1px solid #444; background: #333; color: #ccc; }
  #voiceAudioPlayer { width: 100%; margin: 14px 0; accent-color: #2ecc71; }

  /* ── Toast (delete confirm) ── */
  .toast-overlay {
    display: none; position: fixed; inset: 0; z-index: 500;
    align-items: flex-end; justify-content: center; padding-bottom: 32px; pointer-events: none;
  }
  .toast-overlay.open { display: flex; pointer-events: all; }
  .toast {
    background: linear-gradient(180deg, #2a2a2a, #1a1a1a); border: 1px solid #000; border-radius: 12px;
    padding: 14px 18px; display: flex; align-items: center; gap: 14px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.4); animation: slideUp 0.2s ease;
  }
  @keyframes slideUp { from { transform: translateY(16px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
  .toast-msg { font-size: 13px; color: #fff; font-weight: 500; }
  .toast-msg span { color: #888; font-size: 12px; display: block; margin-top: 1px; }
  .toast-actions { display: flex; gap: 8px; }
  .toast-cancel { padding: 6px 14px; border-radius: 7px; border: 1px solid #444; background: #333; color: #ccc; font-size: 12px; font-family: inherit; cursor: pointer; }
  .toast-confirm { padding: 6px 14px; border-radius: 7px; border: 1px solid #c0392b; background: linear-gradient(180deg, #e74c3c, #c0392b); color: #fff; font-size: 12px; font-weight: 600; font-family: inherit; cursor: pointer; box-shadow: 0 2px 0 #922b21; }

  /* ── Modal ── */
  .modal-overlay {
    display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 200;
    align-items: center; justify-content: center; backdrop-filter: blur(3px);
  }
  .modal-overlay.open { display: flex; }
  .modal {
    background: linear-gradient(180deg, #ffffff 0%, #f5f5f2 100%);
    border-radius: 14px; border: 1px solid #d0d0cc; padding: 28px 30px; width: 460px; max-width: 95vw;
    box-shadow: 0 24px 60px rgba(0,0,0,0.3); position: relative; max-height: 90vh; overflow-y: auto;
  }
  .modal::before { content:''; position:absolute; inset:5px; border-radius:10px; border:1px dashed rgba(0,0,0,0.08); pointer-events:none; }
  .modal-title { font-size: 16px; font-weight: 700; margin-bottom: 22px; color: #1a1a1a; padding-bottom: 14px; border-bottom: 1px solid #e8e8e4; }
  .form-group { margin-bottom: 15px; }
  .form-label { font-size: 11px; font-weight: 700; color: #999; text-transform: uppercase; letter-spacing: 0.08em; margin-bottom: 6px; display: block; }
  .form-input, .form-select, .form-textarea {
    width: 100%; padding: 9px 11px; border: 1px solid #d0d0cc; border-radius: 8px;
    font-size: 13px; font-family: inherit; color: #1a1a1a;
    background: linear-gradient(180deg, #fafaf8 0%, #f5f5f2 100%);
    box-shadow: inset 0 2px 5px rgba(0,0,0,0.07); transition: border-color 0.15s;
  }
  .form-textarea { resize: vertical; min-height: 72px; line-height: 1.5; }
  .form-input:focus, .form-select:focus, .form-textarea:focus { outline: none; border-color: #888; box-shadow: inset 0 2px 5px rgba(0,0,0,0.07), 0 0 0 3px rgba(0,0,0,0.08); }
  .modal-actions { display: flex; gap: 10px; justify-content: flex-end; margin-top: 22px; }
  .btn-cancel { padding: 8px 16px; border-radius: 8px; border: 1px solid #d0d0cc; background: linear-gradient(180deg, #fafaf8, #e8e8e4); font-size: 13px; font-family: inherit; cursor: pointer; color: #666; box-shadow: 0 2px 0 #c0c0bc; }
  .btn-save { padding: 8px 20px; border-radius: 8px; border: 1px solid #000; background: linear-gradient(180deg, #2a2a2a, #1a1a1a); color: #fff; font-size: 13px; font-family: inherit; font-weight: 600; cursor: pointer; box-shadow: 0 3px 0 #000; }
  .btn-save:hover { background: linear-gradient(180deg, #333, #222); }
  .btn-save:active { transform: translateY(2px); box-shadow: 0 1px 0 #000; }

  /* ── Desktop scale ── */
  @media (min-width: 701px) {
    body { zoom: 1.1; }
  }

  /* ── Mobile ── */
  @media (max-width: 700px) {
    body { padding: 0 0 60px; zoom: 1; }
    .app { max-width: 100%; }
    .header { border-radius: 0; padding: 14px 16px; }
    .brand-title { font-size: 14px; }
    .brand-sub { display: none; }
    .user-badge { display: none; }
    .btn-add { padding: 8px 12px; font-size: 12px; }
    .body-panel { border-radius: 0; padding: 16px 12px 24px; border-left: none; border-right: none; }
    .stat-bar { grid-template-columns: repeat(2,1fr); gap: 8px; margin-bottom: 16px; }
    .stat { padding: 10px 12px; }
    .stat-value { font-size: 22px; }
    .filter-row-label { width: 52px; font-size: 9px; }
    .filter-pills { flex-wrap: nowrap; overflow-x: auto; -webkit-overflow-scrolling: touch; padding-bottom: 4px; }
    .filter-pills::-webkit-scrollbar { display: none; }
    .fil-btn, .cat-btn { font-size: 11px; padding: 4px 10px; }
    .table-wrap { border: none; border-radius: 0; box-shadow: none; }
    table, thead, tbody, tr, td, th { display: block; }
    thead { display: none; }
    tr {
      background: linear-gradient(180deg, #fff, #f8f8f5) !important;
      border: 1px solid #d8d8d4; border-radius: 10px; margin-bottom: 10px;
      padding: 12px 14px; box-shadow: 0 2px 8px rgba(0,0,0,0.08); position: relative;
    }
    tr.selected { background: rgba(200,210,255,0.4) !important; border-color: #a0b0f0; }
    tr:last-child { border-bottom: 1px solid #d8d8d4; margin-bottom: 0; }
    tr.done { opacity: 0.5; }
    tr.done .task-name { text-decoration: line-through; }
    td { padding: 0; border: none; background: transparent !important; width: auto !important; white-space: normal !important; }
    td:nth-child(1) { display: inline-block; vertical-align: middle; margin-right: 8px; }
    td:nth-child(2) { display: inline-block; vertical-align: middle; width: calc(100% - 60px) !important; }
    .task-name { font-size: 14px; font-weight: 500; }
    td:nth-child(3), td:nth-child(4), td:nth-child(5) { display: inline-block; margin-top: 10px; margin-right: 6px; vertical-align: middle; }
    td:nth-child(4) .pri-select, td:nth-child(5) .assign-select { min-width: unset; font-size: 11px; padding: 4px 7px; }
    td:nth-child(6) { display: none; }
    td:nth-child(7) { position: absolute; top: 10px; right: 10px; display: flex; gap: 2px; }
    .btn-del, .btn-edit, .btn-voice, .btn-focus { opacity: 0.4 !important; font-size: 17px; padding: 4px; }
    .btn-voice.has-voice { opacity: 1 !important; color: #22c55e; }
    .btn-focus.running { opacity: 1 !important; }
    .empty, .loading { padding: 32px; }
    .expand-row { background: none !important; border: none !important; box-shadow: none !important; padding: 0 !important; margin-top: -10px; margin-bottom: 10px; border-radius: 0 0 10px 10px !important; }
    .expand-row > td { display: block !important; border-radius: 0 0 10px 10px; border: 1px solid #d8d8d4; border-top: none !important; }
    .expand-content { padding: 10px 14px 14px !important; }
    .expand-content .expand-audio { max-width: 100%; }
    .expand-audio { max-width: 100%; }
    .voice-box { min-width: min(320px, 90vw) !important; max-width: 94vw !important; width: 94vw !important; padding: 16px 18px; }
    .focus-timer-display { font-size: 36px; }
    .focus-session-row { flex-wrap: wrap; gap: 4px; }
    .fsd { flex-basis: 100%; }
  }
</style>
</head>
<body>

<!-- ── Lock screen ── -->
<div id="lockScreen" style="position:fixed;inset:0;z-index:9999;background:#e8e8e4;background-image:repeating-linear-gradient(0deg,transparent,transparent 24px,rgba(0,0,0,0.02) 24px,rgba(0,0,0,0.02) 25px),repeating-linear-gradient(90deg,transparent,transparent 24px,rgba(0,0,0,0.015) 24px,rgba(0,0,0,0.015) 25px);display:flex;align-items:center;justify-content:center;">
  <div style="background:linear-gradient(180deg,#fff,#f5f5f2);border:1px solid #d0d0cc;border-radius:14px;padding:36px 40px;width:360px;max-width:92vw;box-shadow:0 24px 60px rgba(0,0,0,0.18);position:relative;">
    <div style="position:absolute;inset:5px;border-radius:10px;border:1px dashed rgba(0,0,0,0.08);pointer-events:none;"></div>
    <div style="display:flex;align-items:center;gap:12px;margin-bottom:28px;">
      <div style="width:34px;height:34px;border-radius:8px;background:linear-gradient(145deg,#2a2a2a,#111);display:flex;align-items:center;justify-content:center;color:#fff;font-size:17px;">
        <i class="ti ti-lock"></i>
      </div>
      <div>
        <div style="font-size:15px;font-weight:700;color:#1a1a1a;">Task Board</div>
        <div style="font-size:11px;color:#888;">Otternative Marketing</div>
      </div>
    </div>
    <label style="font-size:11px;font-weight:700;color:#999;text-transform:uppercase;letter-spacing:0.08em;display:block;margin-bottom:7px;">Password</label>
    <input id="pwInput" type="password" placeholder="Enter password…" onkeydown="if(event.key==='Enter')checkPw()" style="width:100%;padding:10px 12px;border:1px solid #d0d0cc;border-radius:8px;font-size:14px;font-family:inherit;color:#1a1a1a;background:linear-gradient(180deg,#fafaf8,#f5f5f2);box-shadow:inset 0 2px 5px rgba(0,0,0,0.07);outline:none;margin-bottom:10px;" />
    <div id="pwError" style="font-size:12px;color:#a32d2d;margin-bottom:10px;display:none;">Incorrect password. Try again.</div>
    <button onclick="checkPw()" style="width:100%;padding:10px;border-radius:8px;border:1px solid #000;background:linear-gradient(180deg,#2a2a2a,#1a1a1a);color:#fff;font-size:14px;font-family:inherit;font-weight:600;cursor:pointer;box-shadow:0 3px 0 #000;">Unlock</button>
  </div>
</div>

<!-- ── Who are you? screen ── -->
<div id="whoScreen" style="display:none;position:fixed;inset:0;z-index:9998;background:#e8e8e4;background-image:repeating-linear-gradient(0deg,transparent,transparent 24px,rgba(0,0,0,0.02) 24px,rgba(0,0,0,0.02) 25px);align-items:center;justify-content:center;">
  <div style="background:linear-gradient(180deg,#fff,#f5f5f2);border:1px solid #d0d0cc;border-radius:14px;padding:36px 40px;width:360px;max-width:92vw;box-shadow:0 24px 60px rgba(0,0,0,0.18);position:relative;">
    <div style="position:absolute;inset:5px;border-radius:10px;border:1px dashed rgba(0,0,0,0.08);pointer-events:none;"></div>
    <div style="display:flex;align-items:center;gap:12px;margin-bottom:28px;">
      <div style="width:34px;height:34px;border-radius:8px;background:linear-gradient(145deg,#2a2a2a,#111);display:flex;align-items:center;justify-content:center;color:#fff;font-size:17px;">
        <i class="ti ti-user"></i>
      </div>
      <div>
        <div style="font-size:15px;font-weight:700;color:#1a1a1a;">Who are you?</div>
        <div style="font-size:11px;color:#888;">So we can track updates</div>
      </div>
    </div>
    <label style="font-size:11px;font-weight:700;color:#999;text-transform:uppercase;letter-spacing:0.08em;display:block;margin-bottom:7px;">Select your name</label>
    <select id="whoSelect" style="width:100%;padding:10px 12px;border:1px solid #d0d0cc;border-radius:8px;font-size:14px;font-family:inherit;color:#1a1a1a;background:linear-gradient(180deg,#fafaf8,#f5f5f2);box-shadow:inset 0 2px 5px rgba(0,0,0,0.07);outline:none;margin-bottom:16px;">
      <option value="">— Pick your name —</option>
      <option>Ajeeth</option><option>Hari</option><option>Shahzeb</option>
      <option>Narendra</option><option>Angela</option><option>Kira</option><option>Bea</option>
    </select>
    <button onclick="confirmWho()" style="width:100%;padding:10px;border-radius:8px;border:1px solid #000;background:linear-gradient(180deg,#2a2a2a,#1a1a1a);color:#fff;font-size:14px;font-family:inherit;font-weight:600;cursor:pointer;box-shadow:0 3px 0 #000;">Continue</button>
  </div>
</div>

<!-- ── Main app ── -->
<div class="app" id="mainApp" style="display:none">
  <div class="header">
    <div class="brand">
      <div class="brand-icon"><i class="ti ti-layout-kanban"></i></div>
      <div>
        <div class="brand-title">Task Board</div>
        <div class="brand-sub">Otternative Marketing</div>
      </div>
    </div>
    <div class="header-right">
      <div class="user-badge"><i class="ti ti-user" style="font-size:12px"></i> <span id="userLabel"></span></div>
      <button class="btn-add" onclick="openModal()"><i class="ti ti-plus"></i> Add Task</button>
    </div>
  </div>

  <div class="body-panel">
    <div class="stat-bar" id="statBar"></div>

    <div class="search-wrap">
      <i class="ti ti-search"></i>
      <input class="search-input" id="searchInput" placeholder="Search tasks…" oninput="render()" />
    </div>

    <div class="filter-rows">
      <div class="filter-row">
        <span class="filter-row-label">Category</span>
        <div class="filter-pills">
          <button class="cat-btn active" onclick="setFilter('All',this)">All <span class="count-chip" id="cnt-all"></span></button>
          <button class="cat-btn" onclick="setFilter('BMs',this)">BMs <span class="count-chip" id="cnt-bms"></span></button>
          <button class="cat-btn" onclick="setFilter('Ad Accounts',this)">Ad Accounts <span class="count-chip" id="cnt-ad"></span></button>
          <button class="cat-btn" onclick="setFilter('Profiles & Pages',this)">Profiles &amp; Pages <span class="count-chip" id="cnt-pp"></span></button>
          <button class="cat-btn" onclick="setFilter('Performance',this)">Performance <span class="count-chip" id="cnt-perf"></span></button>
          <button class="cat-btn" onclick="setFilter('Client Work',this)">Client Work <span class="count-chip" id="cnt-cw"></span></button>
          <button class="cat-btn" onclick="setFilter('General',this)">General <span class="count-chip" id="cnt-gen"></span></button>
        </div>
      </div>
      <div class="filter-row">
        <span class="filter-row-label">Priority</span>
        <div class="filter-pills">
          <button class="fil-btn active" onclick="setPriFilter('',this)">All</button>
          <button class="fil-btn" onclick="setPriFilter('High',this)">High</button>
          <button class="fil-btn" onclick="setPriFilter('Medium',this)">Medium</button>
          <button class="fil-btn" onclick="setPriFilter('Low',this)">Low</button>
        </div>
      </div>
      <div class="filter-row">
        <span class="filter-row-label">Assignee</span>
        <div class="filter-pills">
          <button class="fil-btn active" onclick="setAssignFilter('',this)">All</button>
          <button class="fil-btn" onclick="setAssignFilter('unassigned',this)">Unassigned</button>
          <button class="fil-btn" onclick="setAssignFilter('Ajeeth',this)">Ajeeth</button>
          <button class="fil-btn" onclick="setAssignFilter('Hari',this)">Hari</button>
          <button class="fil-btn" onclick="setAssignFilter('Shahzeb',this)">Shahzeb</button>
          <button class="fil-btn" onclick="setAssignFilter('Narendra',this)">Narendra</button>
          <button class="fil-btn" onclick="setAssignFilter('Angela',this)">Angela</button>
          <button class="fil-btn" onclick="setAssignFilter('Kira',this)">Kira</button>
          <button class="fil-btn" onclick="setAssignFilter('Bea',this)">Bea</button>
        </div>
      </div>
    </div>

    <div class="table-wrap">
      <table>
        <thead>
          <tr>
            <th style="width:30px"></th>
            <th>Task</th>
            <th>Category</th>
            <th>Priority</th>
            <th>Assignee</th>
            <th>Updated by</th>
            <th style="width:70px"></th>
          </tr>
        </thead>
        <tbody id="taskBody"><tr><td colspan="7"><div class="loading">Loading tasks…</div></td></tr></tbody>
      </table>
    </div>
  </div>
</div>

<!-- ── Delete toast ── -->
<div class="toast-overlay" id="toastOverlay">
  <div class="toast">
    <div class="toast-msg">Delete this task? <span id="toastTaskName"></span></div>
    <div class="toast-actions">
      <button class="toast-cancel" onclick="closeToast()">Cancel</button>
      <button class="toast-confirm" onclick="confirmDelete()">Delete</button>
    </div>
  </div>
</div>

<!-- ── Add / Edit modal ── -->
<div class="modal-overlay" id="modalOverlay">
  <div class="modal">
    <div class="modal-title" id="modalTitle">New Task</div>
    <div class="form-group">
      <label class="form-label">Task name</label>
      <input class="form-input" id="newName" placeholder="Describe the task…" />
    </div>
    <div class="form-group">
      <label class="form-label">Category</label>
      <select class="form-select" id="newCat">
        <option>BMs</option><option>Ad Accounts</option><option>Profiles &amp; Pages</option>
        <option>Performance</option><option>Client Work</option><option>General</option>
      </select>
    </div>
    <div class="form-group">
      <label class="form-label">Priority</label>
      <select class="form-select" id="newPri">
        <option>High</option><option selected>Medium</option><option>Low</option>
      </select>
    </div>
    <div class="form-group">
      <label class="form-label">Assignee</label>
      <select class="form-select" id="newAssign">
        <option value="">Unassigned</option>
        <option>Ajeeth</option><option>Hari</option><option>Shahzeb</option>
        <option>Narendra</option><option>Angela</option><option>Kira</option><option>Bea</option>
      </select>
    </div>
    <div class="form-group">
      <label class="form-label">Notes <span style="font-weight:400;text-transform:none;letter-spacing:0;color:#bbb">(optional)</span></label>
      <textarea class="form-textarea" id="newNotes" placeholder="Any additional context…"></textarea>
    </div>
    <div class="modal-actions">
      <button class="btn-cancel" onclick="closeModal()">Cancel</button>
      <button class="btn-save" id="modalSaveBtn" onclick="saveTask()">Add Task</button>
    </div>
  </div>
</div>

<!-- ── Focus timer overlay ── -->
<div class="voice-overlay" id="focusOverlay">
  <div class="voice-box" style="min-width:min(340px,92vw);max-width:92vw;width:92vw">
    <div style="font-size:12px;font-weight:700;color:#888;letter-spacing:0.08em;text-transform:uppercase;text-align:center;margin-bottom:10px"><i class="ti ti-clock" style="color:#f59e0b"></i> Focus Timer</div>
    <div class="voice-task-name" id="focusTaskName" style="text-align:center;font-size:12px;color:#888;margin-bottom:0"></div>

    <!-- Confirm state -->
    <div id="focusConfirmState">
      <div style="font-size:15px;font-weight:600;color:#fff;text-align:center;margin:16px 0 20px">Start timer for this task?</div>
      <div class="voice-box-actions" style="justify-content:center">
        <button class="vbtn vbtn-cancel" onclick="cancelFocus()">Cancel</button>
        <button class="vbtn vbtn-stop" onclick="confirmStartFocus()"><i class="ti ti-player-play-filled"></i> Start Timer</button>
      </div>
    </div>

    <!-- Running state -->
    <div id="focusRunningState" style="display:none">
      <div class="focus-timer-display" id="focusTimerDisplay">0:00</div>
      <div class="focus-started-by" id="focusStartedBy"></div>
      <div class="voice-box-actions" style="justify-content:center">
        <button class="vbtn vbtn-cancel" onclick="cancelFocus()">Cancel</button>
        <button class="vbtn vbtn-stop" onclick="stopFocus()"><i class="ti ti-player-stop-filled"></i> Stop &amp; Save</button>
      </div>
    </div>
  </div>
</div>

<!-- ── Voice record overlay ── -->
<div class="voice-overlay" id="voiceRecordOverlay">
  <div class="voice-box">
    <div class="voice-task-name" id="voiceRecordTaskName"></div>
    <div class="rec-indicator">
      <span class="rec-dot"></span>
      <span class="rec-timer" id="recTimer">0:00</span>
    </div>
    <div class="voice-box-actions">
      <button class="vbtn vbtn-cancel" onclick="cancelRecording()">Cancel</button>
      <button class="vbtn vbtn-stop" onclick="stopRecording()"><i class="ti ti-check"></i> Stop &amp; Save</button>
    </div>
  </div>
</div>

<!-- ── Voice play overlay ── -->
<div class="voice-overlay" id="voicePlayOverlay">
  <div class="voice-box">
    <div style="font-size:13px;font-weight:600;color:#fff;margin-bottom:4px;"><i class="ti ti-microphone" style="color:#22c55e"></i> Voice Note</div>
    <div class="voice-task-name" id="voicePlayTaskName"></div>
    <audio id="voiceAudioPlayer" controls></audio>
    <div class="voice-box-actions">
      <button class="vbtn vbtn-del" onclick="deleteVoice()"><i class="ti ti-trash"></i> Delete</button>
      <button class="vbtn vbtn-rerec" onclick="reRecord()"><i class="ti ti-refresh"></i> Re-record</button>
      <button class="vbtn vbtn-close" onclick="closeVoicePlay()">Close</button>
    </div>
  </div>
</div>

<script>
const SUPABASE_URL = 'https://kcfueoxyyqrcidzucypd.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtjZnVlb3h5eXFyY2lkenVjeXBkIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODIwMzMzNTAsImV4cCI6MjA5NzYwOTM1MH0.fcD3r3WlGD9Y8uRUhpYLfNgltn-jDnoKymi66ADVqPI';
const db = supabase.createClient(SUPABASE_URL, SUPABASE_KEY);

const PEOPLE    = ['','Ajeeth','Hari','Shahzeb','Narendra','Angela','Kira','Bea'];
const PRIS      = ['High','Medium','Low'];
const CAT_MAP   = {'BMs':'BM','Ad Accounts':'AD','Profiles & Pages':'PP','Performance':'PERF','Client Work':'CW','General':'GEN'};
const CAT_CLASS = {'BMs':'badge-bm','Ad Accounts':'badge-ad','Profiles & Pages':'badge-pp','Performance':'badge-perf','Client Work':'badge-cw','General':'badge-gen'};
const PRI_CLASS = {'High':'pri-high','Medium':'pri-med','Low':'pri-low'};

const SEED = [
  {title:'Remove disabled BMs access from Safe BMs',category:'BMs',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'Remove sellers from all existing BMs',category:'BMs',priority:'High',assignee:'Hari',completed:false},
  {title:'Make sure Kira and Bea join those 9 BMs',category:'BMs',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'Replacement for Unique Tyreoils BM',category:'BMs',priority:'Medium',assignee:null,completed:false},
  {title:'New BM Warmup process',category:'BMs',priority:'Medium',assignee:'Shahzeb',completed:false},
  {title:'Who has access to which BM',category:'BMs',priority:'Medium',assignee:'Hari',completed:false},
  {title:'21 Angela BMs',category:'BMs',priority:'Medium',assignee:'Angela',completed:false},
  {title:'Next steps for Fast Food BM',category:'BMs',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'Give Kira access to all existing BMs',category:'BMs',priority:'High',assignee:'Kira',completed:false},
  {title:'Check email and accept all pending BM invites',category:'BMs',priority:'Medium',assignee:'Shahzeb',completed:false},
  {title:'Paused ad accounts to be taken',category:'Ad Accounts',priority:'High',assignee:'Hari',completed:false},
  {title:'Unsettled ad accounts to be fixed and taken',category:'Ad Accounts',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'Ad accounts in Safe BMs to be assigned to Non-English brands',category:'Ad Accounts',priority:'Medium',assignee:null,completed:false},
  {title:'Check on Aurora ad account',category:'Ad Accounts',priority:'Medium',assignee:'Shahzeb',completed:false},
  {title:'Check existing inventory and assign ad accounts',category:'Ad Accounts',priority:'High',assignee:'Narendra',completed:false},
  {title:'Check profile inventory and make sure we have enough',category:'Profiles & Pages',priority:'Medium',assignee:'Hari',completed:false},
  {title:'Get 50 pages ready',category:'Profiles & Pages',priority:'High',assignee:'Shahzeb',completed:false},
  {title:'Check for spend in unassigned ad accounts',category:'Performance',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'FAPAN check',category:'Performance',priority:'High',assignee:'Narendra',completed:false},
  {title:'Check spend till 17 June and forecast monthly spend',category:'Performance',priority:'High',assignee:'Hari',completed:false},
  {title:'Cross check OTT vs Meta spend data and identify gaps',category:'Performance',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'Crack TikTok through agency ad accounts',category:'Performance',priority:'Medium',assignee:'Shahzeb',completed:false},
  {title:'Ask team to ensure Direct Joinee Meta data matches with Telegram real joinees',category:'Performance',priority:'Medium',assignee:null,completed:false},
  {title:'Ask Angela to fill spend sheet every 5 days',category:'Performance',priority:'Low',assignee:'Angela',completed:false},
  {title:'Inform Bea about the spend forecast and give headsup',category:'Performance',priority:'Medium',assignee:'Bea',completed:false},
  {title:'Dan Microsoft Campaign',category:'Client Work',priority:'High',assignee:'Hari',completed:false},
  {title:'Ben Joinee report',category:'Client Work',priority:'High',assignee:'Shahzeb',completed:false},
  {title:'FF Non English brands launch',category:'Client Work',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'Create AI images and videos for clients for free',category:'Client Work',priority:'Medium',assignee:'Narendra',completed:false},
  {title:'Competitor research for all clients',category:'Client Work',priority:'Medium',assignee:null,completed:false},
  {title:'Make sure all client WhatsApp groups are given an update',category:'Client Work',priority:'Medium',assignee:'Hari',completed:false},
  {title:'Make sure Angela takes care of topups, balances, overdrafts',category:'Client Work',priority:'High',assignee:'Angela',completed:false},
  {title:'Kira expense tracker',category:'General',priority:'Medium',assignee:'Kira',completed:false},
  {title:'Bea case study details',category:'General',priority:'Low',assignee:'Bea',completed:false},
  {title:'Setup new Slack Reminders',category:'General',priority:'Medium',assignee:'Shahzeb',completed:false},
  {title:'Fix existing Slack reminders',category:'General',priority:'Medium',assignee:'Shahzeb',completed:false},
  {title:'Team call',category:'General',priority:'High',assignee:'Ajeeth',completed:false},
  {title:'Quantstone TG account ban issue',category:'General',priority:'High',assignee:'Narendra',completed:false},
  {title:'Make sure everyone uses Task Tracker in Claude',category:'General',priority:'Medium',assignee:null,completed:false},
  {title:'Add all credentials to 1password',category:'General',priority:'High',assignee:'Hari',completed:false},
  {title:'Daily - Check Slack Later tasks',category:'General',priority:'Medium',assignee:null,completed:false},
  {title:'Internal Tool - Ask for update from Hamza',category:'General',priority:'Medium',assignee:null,completed:false},
];

// ── State ──
let tasks = [], curFilter = 'All', curPriFilter = '', curAssignFilter = '', editingId = null, deletingId = null;
let expandedIds = new Set();
let focusSessions = {}, focusTaskId = null, focusStartTime = null, focusInterval = null;
let currentUser = sessionStorage.getItem('tb_user') || '';

// ── Auth flow ──
function checkPw() {
  const val = document.getElementById('pwInput').value;
  if (val === ',+)xBgs3-wKR1C-2qS5p') {
    sessionStorage.setItem('tb_auth', '1');
    document.getElementById('lockScreen').style.display = 'none';
    if (!currentUser) {
      const ws = document.getElementById('whoScreen');
      ws.style.display = 'flex';
    } else {
      showApp();
    }
  } else {
    document.getElementById('pwError').style.display = 'block';
    document.getElementById('pwInput').value = '';
    document.getElementById('pwInput').focus();
  }
}

function confirmWho() {
  const v = document.getElementById('whoSelect').value;
  if (!v) { alert('Please pick your name.'); return; }
  currentUser = v;
  sessionStorage.setItem('tb_user', v);
  document.getElementById('whoScreen').style.display = 'none';
  showApp();
}

function showApp() {
  document.getElementById('mainApp').style.display = '';
  document.getElementById('userLabel').textContent = currentUser || 'Unknown';
  init();
}

// On page load — check existing session
if (sessionStorage.getItem('tb_auth') === '1') {
  document.getElementById('lockScreen').style.display = 'none';
  if (!currentUser) {
    document.getElementById('whoScreen').style.display = 'flex';
  } else {
    showApp();
  }
}

// ── DB row → task ──
function fromRow(r) {
  return {
    id: r.id, name: r.title, cat: r.category, pri: r.priority,
    assign: r.assignee || '', done: r.completed || false,
    notes: r.notes || '', updatedBy: r.updated_by || '',
    updatedAt: r.updated_at || null, voiceUrl: r.voice_note_url || ''
  };
}

function timeAgo(iso) {
  if (!iso) return '';
  const secs = Math.floor((Date.now() - new Date(iso)) / 1000);
  if (secs < 60) return 'just now';
  if (secs < 3600) return Math.floor(secs/60) + 'm ago';
  if (secs < 86400) return Math.floor(secs/3600) + 'h ago';
  if (secs < 604800) return Math.floor(secs/86400) + 'd ago';
  return new Date(iso).toLocaleDateString('en-GB', {day:'numeric',month:'short'});
}

// ── Focus sessions loader ──
async function loadFocusSessions() {
  const { data } = await db.from('focus_sessions').select('*').order('started_at');
  focusSessions = {};
  (data || []).forEach(s => {
    if (!focusSessions[s.task_id]) focusSessions[s.task_id] = [];
    focusSessions[s.task_id].push(s);
  });
}

function formatDur(secs) {
  if (!secs) return '0:00';
  const h = Math.floor(secs/3600), m = Math.floor((secs%3600)/60), s = secs%60;
  if (h > 0) return `${h}:${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
  return `${m}:${String(s).padStart(2,'0')}`;
}

// ── Init ──
async function init() {
  const { data, error } = await db.from('tasks').select('*').order('created_at');
  if (error) { console.error(error); return; }
  if (data.length === 0) {
    const { data: seeded, error: se } = await db.from('tasks').insert(SEED).select();
    if (se) { console.error(se); return; }
    tasks = seeded.map(fromRow);
  } else {
    tasks = data.map(fromRow);
  }
  await loadFocusSessions();
  render();
  db.channel('tasks-rt').on('postgres_changes', { event: '*', schema: 'public', table: 'tasks' }, p => {
    if (p.eventType === 'INSERT') { if (!tasks.find(t=>t.id===p.new.id)) tasks.push(fromRow(p.new)); }
    else if (p.eventType === 'UPDATE') { const i=tasks.findIndex(t=>t.id===p.new.id); if(i!==-1) tasks[i]=fromRow(p.new); }
    else if (p.eventType === 'DELETE') { tasks=tasks.filter(t=>t.id!==p.old.id); }
    render();
  }).subscribe();
}

// ── Render ──
function esc(s){return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');}

function updateCounts(){
  const c={'All':0,'BMs':0,'Ad Accounts':0,'Profiles & Pages':0,'Performance':0,'Client Work':0,'General':0};
  tasks.forEach(t=>{c['All']++;c[t.cat]++;});
  document.getElementById('cnt-all').textContent=c['All'];
  document.getElementById('cnt-bms').textContent=c['BMs'];
  document.getElementById('cnt-ad').textContent=c['Ad Accounts'];
  document.getElementById('cnt-pp').textContent=c['Profiles & Pages'];
  document.getElementById('cnt-perf').textContent=c['Performance'];
  document.getElementById('cnt-cw').textContent=c['Client Work'];
  document.getElementById('cnt-gen').textContent=c['General'];
}

function filtered(){
  const sq=(document.getElementById('searchInput').value||'').trim().toLowerCase();
  return tasks.filter(t=>{
    if(curFilter!=='All'&&t.cat!==curFilter)return false;
    if(curPriFilter&&t.pri!==curPriFilter)return false;
    if(curAssignFilter==='unassigned'&&t.assign)return false;
    if(curAssignFilter&&curAssignFilter!=='unassigned'&&t.assign!==curAssignFilter)return false;
    if(sq&&!t.name.toLowerCase().includes(sq)&&!t.notes.toLowerCase().includes(sq))return false;
    return true;
  });
}

function updateStats(){
  const vis=filtered();
  const total=vis.length,done=vis.filter(t=>t.done).length;
  const high=vis.filter(t=>t.pri==='High'&&!t.done).length;
  const unass=vis.filter(t=>!t.assign&&!t.done).length;
  document.getElementById('statBar').innerHTML=`
    <div class="stat"><div class="stat-label">Total Tasks</div><div class="stat-value">${total}</div></div>
    <div class="stat"><div class="stat-label">Completed</div><div class="stat-value">${done}</div></div>
    <div class="stat"><div class="stat-label">High Priority Open</div><div class="stat-value">${high}</div></div>
    <div class="stat"><div class="stat-label">Unassigned Open</div><div class="stat-value">${unass}</div></div>`;
}

function render(){
  updateCounts(); updateStats();
  const vis=filtered();
  const tbody=document.getElementById('taskBody');
  if(!vis.length){tbody.innerHTML=`<tr><td colspan="7"><div class="empty">No tasks match.</div></td></tr>`;return;}
  tbody.innerHTML=vis.flatMap(t=>{
    const sessions = focusSessions[t.id] || [];
    const totalSecs = sessions.reduce((s,x)=>s+(x.duration_seconds||0),0);
    const hasExtra = t.notes || t.voiceUrl || sessions.length > 0;
    const isOpen = expandedIds.has(t.id);
    const chips = [
      t.notes ? `<span class="detail-chip chip-note"><i class="ti ti-notes"></i> Note</span>` : '',
      t.voiceUrl ? `<span class="detail-chip chip-voice"><i class="ti ti-microphone"></i> Voice</span>` : '',
      sessions.length ? `<span class="detail-chip chip-time"><i class="ti ti-clock"></i> ${formatDur(totalSecs)}</span>` : ''
    ].filter(Boolean).join('');
    const isRunning = focusTaskId === t.id;
    const mainRow = `
    <tr class="${t.done?'done':''}" id="row-${t.id}">
      <td><input type="checkbox" class="cb" ${t.done?'checked':''} onchange="toggleDone(${t.id})"/></td>
      <td>
        <span class="task-name">${esc(t.name)}</span>
        ${hasExtra?`<br><span class="detail-chips ${isOpen?'open':''}" onclick="toggleExpand(${t.id})">${chips}<i class="ti ti-chevron-down detail-chevron"></i></span>`:''}
      </td>
      <td><span class="cat-badge ${CAT_CLASS[t.cat]}">${CAT_MAP[t.cat]}</span></td>
      <td><select class="pri-select ${PRI_CLASS[t.pri]}" onchange="setPri(${t.id},this.value)">${PRIS.map(p=>`<option${p===t.pri?' selected':''}>${p}</option>`).join('')}</select></td>
      <td><select class="assign-select" onchange="setAssign(${t.id},this.value)">${PEOPLE.map(p=>`<option value="${p}"${p===t.assign?' selected':''}>${p||'Unassigned'}</option>`).join('')}</select></td>
      <td><span class="updated-by">${t.updatedBy?`${esc(t.updatedBy)}<br><span style="font-size:10px;color:#ccc">${timeAgo(t.updatedAt)}</span>`:'—'}</span></td>
      <td style="white-space:nowrap">
        <button class="btn-focus ${isRunning?'running':''}" onclick="startFocus(${t.id})" title="Focus timer"><i class="ti ti-clock-play"></i></button>
        <button class="btn-voice ${t.voiceUrl?'has-voice':''}" onclick="${t.voiceUrl?`playVoice(${t.id})`:`startRecording(${t.id})`}" title="${t.voiceUrl?'Play voice note':'Record voice note'}"><i class="ti ti-${t.voiceUrl?'player-play':'microphone'}"></i></button>
        <button class="btn-edit" onclick="openEdit(${t.id})" title="Edit"><i class="ti ti-pencil"></i></button>
        <button class="btn-del" onclick="delTask(${t.id})" title="Delete"><i class="ti ti-trash"></i></button>
      </td>
    </tr>`;
    const expandRow = (hasExtra && isOpen) ? `
    <tr class="expand-row" id="expand-${t.id}">
      <td colspan="7">
        <div class="expand-content">
          ${(t.notes||t.voiceUrl)?`
          <div class="expand-who"><span>${esc(t.updatedBy||'Someone')}</span> added</div>
          ${t.notes?`<div class="expand-notes-text">${esc(t.notes)}</div>`:''}
          ${t.voiceUrl?`<audio class="expand-audio" controls src="${t.voiceUrl}"></audio>`:''}
          `:''}
          ${sessions.length?`
          <div class="focus-sessions-wrap" style="${t.notes||t.voiceUrl?'':'border-top:none;padding-top:0'}" >
            <div class="focus-sessions-header"><i class="ti ti-clock"></i> Focus Sessions</div>
            ${sessions.map(s=>`
            <div class="focus-session-row">
              <span class="fsw">${esc(s.started_by||'?')}</span>
              <span class="fsd">${new Date(s.started_at).toLocaleDateString('en-GB',{day:'numeric',month:'short'})}, ${new Date(s.started_at).toLocaleTimeString('en-GB',{hour:'2-digit',minute:'2-digit'})}</span>
              <span class="fsdur">${formatDur(s.duration_seconds||0)}</span>
            </div>`).join('')}
            <div class="focus-total-row"><span>Total time</span><span>${formatDur(totalSecs)}</span></div>
          </div>`:''}
        </div>
      </td>
    </tr>` : '';
    return [mainRow, expandRow];
  }).join('');
}

function toggleExpand(id){
  if(expandedIds.has(id)) expandedIds.delete(id); else expandedIds.add(id);
  render();
}

// ── Single actions ──
function setFilter(cat,btn){curFilter=cat;document.querySelectorAll('.cat-btn').forEach(b=>b.classList.remove('active'));btn.classList.add('active');render();}
function setPriFilter(v,btn){curPriFilter=v;btn.closest('.filter-pills').querySelectorAll('.fil-btn').forEach(b=>b.classList.remove('active'));btn.classList.add('active');render();}
function setAssignFilter(v,btn){curAssignFilter=v;btn.closest('.filter-pills').querySelectorAll('.fil-btn').forEach(b=>b.classList.remove('active'));btn.classList.add('active');render();}

function now(){ return new Date().toISOString(); }

async function toggleDone(id){
  const t=tasks.find(x=>x.id===id); if(!t)return;
  await db.from('tasks').update({completed:!t.done,updated_by:currentUser,updated_at:now()}).eq('id',id);
}
async function setPri(id,v){ await db.from('tasks').update({priority:v,updated_by:currentUser,updated_at:now()}).eq('id',id); }
async function setAssign(id,v){ await db.from('tasks').update({assignee:v||null,updated_by:currentUser,updated_at:now()}).eq('id',id); }

function delTask(id){
  const t=tasks.find(x=>x.id===id); if(!t)return;
  deletingId=id;
  document.getElementById('toastTaskName').textContent=t.name.length>40?t.name.slice(0,40)+'…':t.name;
  document.getElementById('toastOverlay').classList.add('open');
}
function closeToast(){ document.getElementById('toastOverlay').classList.remove('open'); deletingId=null; }
async function confirmDelete(){
  if(!deletingId)return;
  await db.from('tasks').delete().eq('id',deletingId);
  closeToast();
}

// ── Modal ──
function openModal(){
  editingId=null;
  document.getElementById('modalTitle').textContent='New Task';
  document.getElementById('modalSaveBtn').textContent='Add Task';
  document.getElementById('newName').value='';
  document.getElementById('newCat').value='BMs';
  document.getElementById('newPri').value='Medium';
  document.getElementById('newAssign').value='';
  document.getElementById('newNotes').value='';
  document.getElementById('modalOverlay').classList.add('open');
  setTimeout(()=>document.getElementById('newName').focus(),50);
}
function openEdit(id){
  const t=tasks.find(x=>x.id===id); if(!t)return;
  editingId=id;
  document.getElementById('modalTitle').textContent='Edit Task';
  document.getElementById('modalSaveBtn').textContent='Save Changes';
  document.getElementById('newName').value=t.name;
  document.getElementById('newCat').value=t.cat;
  document.getElementById('newPri').value=t.pri;
  document.getElementById('newAssign').value=t.assign||'';
  document.getElementById('newNotes').value=t.notes||'';
  document.getElementById('modalOverlay').classList.add('open');
  setTimeout(()=>document.getElementById('newName').focus(),50);
}
function closeModal(){ document.getElementById('modalOverlay').classList.remove('open'); editingId=null; }

async function saveTask(){
  const name=document.getElementById('newName').value.trim(); if(!name)return;
  const payload={
    title:name, category:document.getElementById('newCat').value,
    priority:document.getElementById('newPri').value,
    assignee:document.getElementById('newAssign').value||null,
    notes:document.getElementById('newNotes').value.trim()||null,
    updated_by:currentUser,
    updated_at:now()
  };
  if(editingId!==null){ await db.from('tasks').update(payload).eq('id',editingId); }
  else { await db.from('tasks').insert({...payload,completed:false}); }
  closeModal();
}

document.getElementById('modalOverlay').addEventListener('click',function(e){if(e.target===this)closeModal();});
document.addEventListener('keydown',e=>{if(e.key==='Escape'){closeModal();closeToast();cancelRecording();closeVoicePlay();cancelFocus();}});

// ── Focus timer ──
function startFocus(id) {
  if (focusTaskId !== null) { alert('A focus timer is already running. Stop it first.'); return; }
  const t = tasks.find(x => x.id === id); if (!t) return;
  focusTaskId = id;
  // Show confirm state
  document.getElementById('focusTaskName').textContent = t.name.length > 55 ? t.name.slice(0,55)+'…' : t.name;
  document.getElementById('focusConfirmState').style.display = '';
  document.getElementById('focusRunningState').style.display = 'none';
  document.getElementById('focusOverlay').classList.add('open');
}

function confirmStartFocus() {
  const t = tasks.find(x => x.id === focusTaskId); if (!t) return;
  focusStartTime = new Date();
  let elapsed = 0;
  document.getElementById('focusStartedBy').textContent = `Started by ${currentUser || 'you'}`;
  document.getElementById('focusTimerDisplay').textContent = '0:00';
  document.getElementById('focusConfirmState').style.display = 'none';
  document.getElementById('focusRunningState').style.display = '';
  render();
  focusInterval = setInterval(() => {
    elapsed++;
    document.getElementById('focusTimerDisplay').textContent = formatDur(elapsed);
  }, 1000);
}

async function stopFocus() {
  if (focusTaskId === null) return;
  clearInterval(focusInterval);
  const duration = Math.max(1, Math.round((new Date() - focusStartTime) / 1000));
  document.getElementById('focusOverlay').classList.remove('open');
  await db.from('focus_sessions').insert({
    task_id: focusTaskId,
    started_by: currentUser || 'Unknown',
    started_at: focusStartTime.toISOString(),
    duration_seconds: duration
  });
  focusTaskId = null; focusStartTime = null; focusInterval = null;
  await loadFocusSessions();
  render();
}

function cancelFocus() {
  if (focusInterval) clearInterval(focusInterval);
  document.getElementById('focusOverlay').classList.remove('open');
  document.getElementById('focusConfirmState').style.display = '';
  document.getElementById('focusRunningState').style.display = 'none';
  focusTaskId = null; focusStartTime = null; focusInterval = null;
  render();
}

// ── Voice notes ──
let voiceTaskId = null, mediaRecorder = null, audioChunks = [], recInterval = null, recSeconds = 0;

function startRecording(id) {
  const t = tasks.find(x => x.id === id); if (!t) return;
  if (!navigator.mediaDevices) { alert('Your browser does not support audio recording. Please use Chrome.'); return; }
  voiceTaskId = id;
  navigator.mediaDevices.getUserMedia({ audio: true }).then(stream => {
    audioChunks = []; recSeconds = 0;
    mediaRecorder = new MediaRecorder(stream);
    mediaRecorder.ondataavailable = e => { if (e.data.size > 0) audioChunks.push(e.data); };
    mediaRecorder.onstop = handleRecordingStop;
    mediaRecorder.start();
    document.getElementById('voiceRecordTaskName').textContent = t.name.length > 55 ? t.name.slice(0, 55) + '…' : t.name;
    document.getElementById('recTimer').textContent = '0:00';
    document.getElementById('voiceRecordOverlay').classList.add('open');
    recInterval = setInterval(() => {
      recSeconds++;
      const m = Math.floor(recSeconds / 60), s = recSeconds % 60;
      document.getElementById('recTimer').textContent = `${m}:${s.toString().padStart(2,'0')}`;
    }, 1000);
  }).catch(() => alert('Microphone access denied. Please allow microphone in your browser settings.'));
}

function stopRecording() {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') mediaRecorder.stop();
  mediaRecorder.stream.getTracks().forEach(t => t.stop());
  clearInterval(recInterval);
  document.getElementById('voiceRecordOverlay').classList.remove('open');
}

function cancelRecording() {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.ondataavailable = null; mediaRecorder.onstop = null;
    mediaRecorder.stop(); mediaRecorder.stream.getTracks().forEach(t => t.stop());
  }
  clearInterval(recInterval);
  document.getElementById('voiceRecordOverlay').classList.remove('open');
  voiceTaskId = null; audioChunks = [];
}

async function handleRecordingStop() {
  const blob = new Blob(audioChunks, { type: 'audio/webm' });
  const fileName = `${voiceTaskId}-${Date.now()}.webm`;
  // Delete old voice note file if exists
  const t = tasks.find(x => x.id === voiceTaskId);
  if (t && t.voiceUrl) {
    const oldName = t.voiceUrl.split('/voice-notes/')[1];
    if (oldName) await db.storage.from('voice-notes').remove([oldName]);
  }
  // Upload new
  const { error: upErr } = await db.storage.from('voice-notes').upload(fileName, blob, { contentType: 'audio/webm', upsert: true });
  if (upErr) { alert('Upload failed: ' + upErr.message); return; }
  const { data: urlData } = db.storage.from('voice-notes').getPublicUrl(fileName);
  await db.from('tasks').update({ voice_note_url: urlData.publicUrl, updated_by: currentUser, updated_at: now() }).eq('id', voiceTaskId);
  voiceTaskId = null; audioChunks = [];
}

function playVoice(id) {
  const t = tasks.find(x => x.id === id); if (!t || !t.voiceUrl) return;
  voiceTaskId = id;
  document.getElementById('voicePlayTaskName').textContent = t.name.length > 55 ? t.name.slice(0, 55) + '…' : t.name;
  const player = document.getElementById('voiceAudioPlayer');
  player.src = t.voiceUrl; player.load();
  document.getElementById('voicePlayOverlay').classList.add('open');
}

function closeVoicePlay() {
  document.getElementById('voiceAudioPlayer').pause();
  document.getElementById('voicePlayOverlay').classList.remove('open');
  voiceTaskId = null;
}

async function deleteVoice() {
  const t = tasks.find(x => x.id === voiceTaskId); if (!t) return;
  const fileName = t.voiceUrl.split('/voice-notes/')[1];
  if (fileName) await db.storage.from('voice-notes').remove([fileName]);
  await db.from('tasks').update({ voice_note_url: null, updated_by: currentUser, updated_at: now() }).eq('id', voiceTaskId);
  closeVoicePlay();
}

function reRecord() {
  const id = voiceTaskId;
  closeVoicePlay();
  setTimeout(() => startRecording(id), 100);
}
</script>
</body>
</html>
