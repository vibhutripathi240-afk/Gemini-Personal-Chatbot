```html
<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Hydrogen Chat — Multi-AI Personal Assistant</title>
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'system-ui', 'sans-serif'],
            mono: ['JetBrains Mono', 'monospace'],
          },
        },
      },
    };
  </script>
  <style>
    * { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; padding: 0; overflow: hidden; }
    .custom-scroll::-webkit-scrollbar { width: 6px; height: 6px; }
    .custom-scroll::-webkit-scrollbar-track { background: transparent; }
    .custom-scroll::-webkit-scrollbar-thumb { background: rgba(148, 163, 184, 0.3); border-radius: 9999px; }
    .custom-scroll::-webkit-scrollbar-thumb:hover { background: rgba(148, 163, 184, 0.5); }
    .prose-msg p { margin: 0 0 0.65rem 0; line-height: 1.6; }
    .prose-msg p:last-child { margin-bottom: 0; }
    .prose-msg ul, .prose-msg ol { margin: 0.4rem 0 0.65rem 1.25rem; }
    .prose-msg li { margin-bottom: 0.25rem; line-height: 1.55; }
    .prose-msg ul { list-style-type: disc; }
    .prose-msg ol { list-style-type: decimal; }
    .prose-msg code.inline-code {
      font-family: 'JetBrains Mono', monospace;
      font-size: 0.85em;
      padding: 0.15rem 0.35rem;
      border-radius: 0.375rem;
      background: rgba(148, 163, 184, 0.18);
    }
    @keyframes pulseDot {
      0%, 80%, 100% { transform: scale(0.6); opacity: 0.4; }
      40% { transform: scale(1); opacity: 1; }
    }
    .dot-1 { animation: pulseDot 1.2s infinite ease-in-out; }
    .dot-2 { animation: pulseDot 1.2s 0.2s infinite ease-in-out; }
    .dot-3 { animation: pulseDot 1.2s 0.4s infinite ease-in-out; }
  </style>
</head>
<body class="bg-slate-50 text-slate-900 dark:bg-[#0d1117] dark:text-slate-100 font-sans antialiased">
  <div id="app" class="flex h-full w-full relative overflow-hidden">
    <!-- Mobile Sidebar Backdrop -->
    <div id="sidebar-backdrop" class="fixed inset-0 bg-black/50 backdrop-blur-xs z-30 hidden md:hidden transition-opacity"></div>

    <!-- Sidebar -->
    <aside id="sidebar" class="fixed md:static inset-y-0 left-0 z-40 w-72 bg-white dark:bg-[#161b22] border-r border-slate-200 dark:border-slate-800 flex flex-col transform -translate-x-full md:translate-x-0 transition-transform duration-200 ease-out shrink-0">
      <!-- Sidebar Header -->
      <div class="p-3.5 border-b border-slate-200 dark:border-slate-800 flex items-center justify-between gap-2">
        <button id="new-chat-btn" class="flex-1 flex items-center justify-center gap-2 px-3.5 py-2.5 rounded-xl bg-sky-600 hover:bg-sky-500 active:bg-sky-700 text-white font-medium text-sm shadow-sm transition-colors cursor-pointer">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15"/></svg>
          <span>New Chat</span>
        </button>
        <button id="close-sidebar-btn" class="md:hidden p-2 rounded-lg text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-800" aria-label="Close sidebar">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12"/></svg>
        </button>
      </div>

      <!-- Chat History List -->
      <div class="flex-1 overflow-y-auto custom-scroll p-2.5 space-y-1" id="chat-list"></div>

      <!-- Sidebar Footer -->
      <div class="p-3 border-t border-slate-200 dark:border-slate-800 space-y-2">
        <div id="mode-indicator" class="px-3 py-2 rounded-xl bg-slate-100 dark:bg-slate-800/70 flex items-center justify-between text-xs">
          <div class="flex items-center gap-2 min-w-0">
            <span id="mode-dot" class="w-2 h-2 rounded-full bg-amber-400 shrink-0"></span>
            <span id="mode-label" class="font-medium truncate text-slate-700 dark:text-slate-300">Preview Mode</span>
          </div>
          <span id="ctx-badge" class="text-[10px] px-1.5 py-0.5 rounded bg-sky-500/15 text-sky-600 dark:text-sky-400 font-semibold shrink-0">1M ctx</span>
        </div>
        <button id="open-settings-btn" class="w-full flex items-center justify-between px-3 py-2.5 rounded-xl hover:bg-slate-100 dark:hover:bg-slate-800 text-sm font-medium text-slate-700 dark:text-slate-200 transition-colors cursor-pointer">
          <span class="flex items-center gap-2.5">
            <svg class="w-4 h-4 text-slate-500" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/><path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/></svg>
            <span>API Keys &amp; Persona</span>
          </span>
          <svg class="w-4 h-4 text-slate-400" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M9 5l7 7-7 7"/></svg>
        </button>
      </div>
    </aside>

    <!-- Main Chat Area -->
    <main class="flex-1 flex flex-col min-w-0 h-full bg-slate-50 dark:bg-[#0d1117]">
      <!-- Top Header -->
      <header class="h-14 px-3 sm:px-5 border-b border-slate-200 dark:border-slate-800 bg-white/80 dark:bg-[#161b22]/80 backdrop-blur-md flex items-center justify-between gap-2 shrink-0">
        <div class="flex items-center gap-2 min-w-0">
          <button id="open-sidebar-btn" class="p-2 -ml-1 rounded-lg text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-800 md:hidden cursor-pointer" aria-label="Open menu">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M4 6h16M4 12h16M4 18h16"/></svg>
          </button>
          <div class="flex items-center gap-2 min-w-0">
            <div id="header-brand-icon" class="w-8 h-8 rounded-xl bg-gradient-to-br from-sky-500 to-indigo-600 flex items-center justify-center text-white shadow-sm shrink-0">
              <svg class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2">
                <circle cx="12" cy="12" r="3"/>
                <ellipse cx="12" cy="12" rx="9" ry="3.8" transform="rotate(35 12 12)"/>
                <ellipse cx="12" cy="12" rx="9" ry="3.8" transform="rotate(-35 12 12)"/>
              </svg>
            </div>
            <!-- Model Switcher -->
            <div class="flex items-center gap-1.5 min-w-0 bg-slate-100 dark:bg-slate-800/90 border border-slate-200 dark:border-slate-700 rounded-xl px-2.5 py-1.5">
              <select id="model-select" class="bg-transparent text-xs sm:text-sm font-semibold text-slate-800 dark:text-slate-100 focus:outline-none cursor-pointer max-w-[160px] sm:max-w-[230px] truncate" title="Switch AI Model">
                <optgroup label="Google Gemini" class="dark:bg-slate-900">
                  <option value="gemini:gemini-4-hydrogen" class="dark:bg-slate-800">Gemini 4 Hydrogen</option>
                  <option value="gemini:gemini-4-neon" class="dark:bg-slate-800">Gemini 4 Neon</option>
                </optgroup>
                <optgroup label="OpenAI ChatGPT" class="dark:bg-slate-900">
                  <option value="openai:gpt-6.2-sol" class="dark:bg-slate-800">ChatGPT — GPT-6.2 Sol</option>
                  <option value="openai:gpt-6.1-terra" class="dark:bg-slate-800">ChatGPT — GPT-6.1 Terra</option>
                  <option value="openai:gpt-6-luna" class="dark:bg-slate-800">ChatGPT — GPT-6 Luna</option>
                </optgroup>
                <optgroup label="Anthropic Claude" class="dark:bg-slate-900">
                  <option value="claude:claude-sonnet-5-5" class="dark:bg-slate-800">Claude Sonnet 5.5</option>
                  <option value="claude:claude-opus-5-5" class="dark:bg-slate-800">Claude Opus 5.5</option>
                  <option value="claude:claude-haiku-5-5" class="dark:bg-slate-800">Claude Haiku 5.5</option>
                </optgroup>
              </select>
            </div>
          </div>
        </div>

        <div class="flex items-center gap-1.5 sm:gap-2 shrink-0">
          <!-- Thinking Level Selector -->
          <div id="thinking-wrap" class="flex items-center gap-1 bg-slate-100 dark:bg-slate-800 rounded-lg px-2 py-1 border border-slate-200 dark:border-slate-700">
            <span class="text-[11px] text-slate-500 dark:text-slate-400 hidden sm:inline">Think:</span>
            <select id="thinking-select" class="bg-transparent text-xs font-medium text-slate-700 dark:text-slate-200 focus:outline-none cursor-pointer" title="Thinking / Reasoning Level">
              <option value="low" class="dark:bg-slate-800">Low</option>
              <option value="medium" class="dark:bg-slate-800">Medium</option>
              <option value="high" selected class="dark:bg-slate-800">High</option>
              <option value="xhigh" class="dark:bg-slate-800">xHigh</option>
            </select>
          </div>

          <!-- Export Button -->
          <button id="export-btn" class="p-2 rounded-lg text-slate-500 hover:text-slate-800 dark:text-slate-400 dark:hover:text-slate-100 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors cursor-pointer" title="Export chat as Markdown">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/></svg>
          </button>

          <!-- Theme Toggle -->
          <button id="theme-btn" class="p-2 rounded-lg text-slate-500 hover:text-slate-800 dark:text-slate-400 dark:hover:text-slate-100 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors cursor-pointer" title="Toggle Dark/Light Theme">
            <svg id="sun-icon" class="w-4 h-4 hidden dark:block" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"/></svg>
            <svg id="moon-icon" class="w-4 h-4 block dark:hidden" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"/></svg>
          </button>
        </div>
      </header>

      <!-- Messages Container -->
      <div id="messages-container" class="flex-1 overflow-y-auto custom-scroll px-3 sm:px-6 py-4">
        <div id="messages-inner" class="max-w-3xl mx-auto space-y-5 pb-4"></div>
      </div>

      <!-- Composer Bar -->
      <div class="p-3 sm:p-4 bg-gradient-to-t from-slate-50 via-slate-50 to-transparent dark:from-[#0d1117] dark:via-[#0d1117] shrink-0">
        <div class="max-w-3xl mx-auto">
          <!-- Attachment Preview -->
          <div id="attachment-bar" class="hidden mb-2 flex items-center gap-2 flex-wrap"></div>

          <!-- Input Box -->
          <div class="flex items-end gap-2 bg-white dark:bg-[#161b22] border border-slate-200 dark:border-slate-700/80 rounded-2xl p-2 shadow-sm focus-within:ring-2 focus-within:ring-sky-500/40 focus-within:border-sky-500 transition-all">
            <label for="file-input" class="p-2 rounded-xl text-slate-500 hover:text-sky-600 dark:text-slate-400 dark:hover:text-sky-400 hover:bg-slate-100 dark:hover:bg-slate-800 cursor-pointer shrink-0 transition-colors" title="Attach image">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486L20.5 13"/></svg>
              <input id="file-input" type="file" accept="image/*" class="hidden" />
            </label>

            <textarea id="prompt-input" rows="1" placeholder="Message Gemini 4 Hydrogen..." class="flex-1 bg-transparent resize-none max-h-36 py-2 px-1 text-sm sm:text-base text-slate-900 dark:text-slate-100 placeholder-slate-400 focus:outline-none leading-relaxed"></textarea>

            <button id="send-btn" class="p-2.5 rounded-xl bg-sky-600 hover:bg-sky-500 disabled:opacity-40 disabled:hover:bg-sky-600 text-white transition-colors shrink-0 cursor-pointer" title="Send message">
              <svg id="send-icon" class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M5 12h14M12 5l7 7-7 7"/></svg>
              <svg id="stop-icon" class="w-4 h-4 hidden" fill="currentColor" viewBox="0 0 24 24"><rect x="6" y="6" width="12" height="12" rx="2"/></svg>
            </button>
          </div>
          <div class="mt-1.5 flex items-center justify-between px-1 text-[11px] text-slate-400 dark:text-slate-500">
            <span id="footer-status" class="truncate">Preview Mode active — add API keys in Settings for live Gemini, ChatGPT, or Claude.</span>
            <span id="token-counter" class="hidden sm:inline font-mono shrink-0 ml-2">0 / 1,048,576 tokens</span>
          </div>
        </div>
      </div>
    </main>
  </div>

  <!-- Settings Modal -->
  <div id="settings-modal" class="fixed inset-0 z-50 bg-black/60 backdrop-blur-xs hidden items-center justify-center p-4">
    <div class="bg-white dark:bg-[#161b22] border border-slate-200 dark:border-slate-700 rounded-2xl w-full max-w-md shadow-2xl overflow-hidden">
      <div class="px-5 py-4 border-b border-slate-200 dark:border-slate-800 flex items-center justify-between">
        <h2 class="font-semibold text-base">API Keys &amp; Persona</h2>
        <button id="close-settings-btn" class="p-1.5 rounded-lg text-slate-400 hover:text-slate-700 dark:hover:text-slate-200 hover:bg-slate-100 dark:hover:bg-slate-800 cursor-pointer">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12"/></svg>
        </button>
      </div>
      <div class="p-5 space-y-4 max-h-[75vh] overflow-y-auto custom-scroll">
        <!-- Gemini API Key -->
        <div>
          <div class="flex items-center justify-between mb-1.5">
            <label for="gemini-key-input" class="text-xs font-semibold uppercase tracking-wider text-sky-600 dark:text-sky-400">Gemini API Key (Free Tier)</label>
            <a href="https://aistudio.google.com/apikey" target="_blank" rel="noopener noreferrer" class="text-xs text-sky-600 dark:text-sky-400 hover:underline">Get key &rarr;</a>
          </div>
          <div class="flex items-center gap-2">
            <input id="gemini-key-input" type="password" placeholder="AIza... (gemini-4-hydrogen, gemini-4-neon)" class="flex-1 px-3 py-2 rounded-xl bg-slate-100 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-sky-500" />
            <button type="button" data-toggle-input="gemini-key-input" class="toggle-vis-btn px-3 py-2 rounded-xl bg-slate-100 dark:bg-slate-800 text-xs font-medium text-slate-600 dark:text-slate-300 hover:bg-slate-200 dark:hover:bg-slate-700 cursor-pointer">Show</button>
          </div>
        </div>

        <!-- OpenAI API Key -->
        <div>
          <div class="flex items-center justify-between mb-1.5">
            <label for="openai-key-input" class="text-xs font-semibold uppercase tracking-wider text-emerald-600 dark:text-emerald-400">OpenAI API Key (ChatGPT)</label>
            <a href="https://platform.openai.com/api-keys" target="_blank" rel="noopener noreferrer" class="text-xs text-emerald-600 dark:text-emerald-400 hover:underline">Get key &rarr;</a>
          </div>
          <div class="flex items-center gap-2">
            <input id="openai-key-input" type="password" placeholder="sk-... (gpt-6.2-sol, terra, luna)" class="flex-1 px-3 py-2 rounded-xl bg-slate-100 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500" />
            <button type="button" data-toggle-input="openai-key-input" class="toggle-vis-btn px-3 py-2 rounded-xl bg-slate-100 dark:bg-slate-800 text-xs font-medium text-slate-600 dark:text-slate-300 hover:bg-slate-200 dark:hover:bg-slate-700 cursor-pointer">Show</button>
          </div>
        </div>

        <!-- Claude API Key -->
        <div>
          <div class="flex items-center justify-between mb-1.5">
            <label for="claude-key-input" class="text-xs font-semibold uppercase tracking-wider text-amber-600 dark:text-amber-400">Anthropic API Key (Claude)</label>
            <a href="https://platform.claude.com/" target="_blank" rel="noopener noreferrer" class="text-xs text-amber-600 dark:text-amber-400 hover:underline">Get key &rarr;</a>
          </div>
          <div class="flex items-center gap-2">
            <input id="claude-key-input" type="password" placeholder="sk-ant-... (claude-sonnet-5-5, opus, haiku)" class="flex-1 px-3 py-2 rounded-xl bg-slate-100 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-amber-500" />
            <button type="button" data-toggle-input="claude-key-input" class="toggle-vis-btn px-3 py-2 rounded-xl bg-slate-100 dark:bg-slate-800 text-xs font-medium text-slate-600 dark:text-slate-300 hover:bg-slate-200 dark:hover:bg-slate-700 cursor-pointer">Show</button>
          </div>
          <p class="mt-1.5 text-[11px] text-slate-500 dark:text-slate-400">Keys stay in your browser only. Leave any key empty to use Preview Mode for that provider.</p>
        </div>

        <!-- Persona Presets -->
        <div>
          <label class="block text-xs font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400 mb-1.5">Persona Preset</label>
          <div class="grid grid-cols-2 gap-2" id="persona-grid">
            <button type="button" data-persona="default" class="persona-btn px-3 py-2 rounded-xl border text-left text-xs font-medium transition-colors cursor-pointer">
              <div class="font-semibold">Personal Assistant</div>
              <div class="text-[11px] opacity-70">Balanced &amp; helpful</div>
            </button>
            <button type="button" data-persona="coder" class="persona-btn px-3 py-2 rounded-xl border text-left text-xs font-medium transition-colors cursor-pointer">
              <div class="font-semibold">Coding Pro</div>
              <div class="text-[11px] opacity-70">Clean code &amp; debug</div>
            </button>
            <button type="button" data-persona="hinglish" class="persona-btn px-3 py-2 rounded-xl border text-left text-xs font-medium transition-colors cursor-pointer">
              <div class
