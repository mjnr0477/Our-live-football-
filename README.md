<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Our Football Live</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    /* Minimal, modern styling */
    body { font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; margin:0; background:#0f1724; color:#e6eef8; }
    header { padding:20px; text-align:center; background:linear-gradient(90deg,#0b1220,#102033); box-shadow:0 6px 24px rgba(0,0,0,0.6); }
    h1 { margin:0; font-size:20px; letter-spacing:0.6px; }
    .wrap { max-width:1100px; margin:24px auto; padding: 0 20px; display:grid; grid-template-columns: 360px 1fr; gap:20px; }
    .card { background:#0b1220; border-radius:12px; padding:14px; box-shadow:0 6px 18px rgba(2,6,23,0.6); }
    input[type="search"] { width:100%; padding:10px 12px; border-radius:8px; border:1px solid #16324a; background:transparent; color:inherit; outline:none; }
    button { margin-top:10px; padding:10px 12px; border-radius:8px; border:0; background:#1f7aeb; color:white; cursor:pointer; }
    .results { margin-top:12px; display:flex; flex-direction:column; gap:8px; max-height:60vh; overflow:auto; }
    .resultItem { display:flex; gap:10px; align-items:center; padding:8px; border-radius:8px; background:rgba(255,255,255,0.02); cursor:pointer; }
    .resultItem img { width:120px; height:68px; object-fit:cover; border-radius:6px; }
    iframe.player { width:100%; height:60vh; border-radius:12px; border:0; background:black; }
    .small { font-size:13px; color:#b7c9d9; }
    .footer { text-align:center; margin:18px 0 40px; color:#9fb4cf; font-size:13px; }
    .link { color:#9be; text-decoration:underline; cursor:pointer; }
    @media (max-width:880px){ .wrap{ grid-template-columns: 1fr; } iframe.player { height:45vh; } }
  </style>
</head>
<body>
  <header>
    <h1>Our Football Live — legal stream aggregator</h1>
    <div class="small">Search public live streams (YouTube). Add official sources you trust.</div>
  </header>

  <main class="wrap">
    <section class="card">
      <label for="q" class="small">Search live streams on YouTube (public channels only)</label>
      <input id="q" type="search" placeholder="Search: e.g. football live, UEFA, highlights" value="football live" />
      <button id="searchBtn">Search YouTube Live</button>

      <div style="margin-top:14px;">
        <label class="small">Add an official stream URL (YouTube/Twitch/embed)</label>
        <input id="manualUrl" placeholder="Paste official stream URL or embed link" />
        <button id="addBtn">Add to list</button>
      </div>

      <div class="results card" id="results" style="margin-top:12px; padding:10px;">
        <!-- search results appear here -->
      </div>
    </section>

    <section>
      <div class="card">
        <div id="playerContainer">
          <div class="small" style="margin-bottom:8px;">Selected stream (click a result to play)</div>
          <!-- Default: show helpful tips -->
          <div id="placeholder" class="small" style="padding:14px; background:rgba(255,255,255,0.02); border-radius:10px;">
            No stream loaded. Try "Search YouTube Live" or add an official stream URL.
          </div>
        </div>
      </div>

      <div class="card" style="margin-top:16px;">
        <div class="small">Notes & Legal</div>
        <ul style="color:#9fb4cf; font-size:13px;">
          <li>Only public, freely available streams will show via YouTube search.</li>
          <li>This app does NOT provide access to paid/copyrighted streams.</li>
          <li>Use this to aggregate official broadcaster channels, federation feeds, or free streams.</li>
          <li>If you run a legal broadcast channel, add your official link in the app.</li>
        </ul>
      </div>
    </section>
  </main>

  <div class="footer">Built with ❤️ — copy this file and run as a static site. Replace <code>YOUR_YOUTUBE_API_KEY</code> with your API key.</div>

  <script>
    /**************************************************************************
     * IMPORTANT:
     * - Replace 'YOUR_YOUTUBE_API_KEY' with a valid YouTube Data API v3 key.
     *   Get one here: https://console.cloud.google.com/apis/credentials
     *
     * - Twitch: embedding Twitch channels works via iframe but finding/searching
     *   Twitch streams requires the Twitch API and OAuth Client credentials.
     *   This sample focuses on YouTube Live search (public, free).
     *
     * LEGAL: Do not use this app to share pirated or paid streams without permission.
     **************************************************************************/

    const YT_API_KEY = 'YOUR_YOUTUBE_API_KEY'; // <<-- INSERT YOUR KEY
    const resultsEl = document.getElementById('results');
    const qEl = document.getElementById('q');
    const searchBtn = document.getElementById('searchBtn');
    const playerContainer = document.getElementById('playerContainer');
    const placeholder = document.getElementById('placeholder');
    const addBtn = document.getElementById('addBtn');
    const manualUrl = document.getElementById('manualUrl');

    function createResultItem(item) {
      const div = document.createElement('div');
      div.className = 'resultItem';
      const thumb = document.createElement('img');
      thumb.src = item.thumbnail;
      const meta = document.createElement('div');
      meta.style.flex = '1';
      const title = document.createElement('div');
      title.textContent = item.title;
      const small = document.createElement('div');
      small.className = 'small';
      small.textContent = item.channel + ' • ' + new Date(item.publishedAt).toLocaleString();
      meta.appendChild(title);
      meta.appendChild(small);
      div.appendChild(thumb);
      div.appendChild(meta);
      div.addEventListener('click', () => playYouTube(item.videoId));
      return div;
    }

    function clearResults() { resultsEl.innerHTML = ''; }

    async function searchYouTubeLive(query) {
      if (YT_API_KEY === 'YOUR_YOUTUBE_API_KEY') {
        alert('Please replace YOUR_YOUTUBE_API_KEY in the file with a valid YouTube Data API key.');
        return;
      }
      clearResults();
      const q = encodeURIComponent(query || 'football live');
      // Search for live events
      const url = `https://www.googleapis.com/youtube/v3/search?part=snippet&type=video&eventType=live&maxResults=12&q=${q}&key=${YT_API_KEY}`;
      try {
        const resp = await fetch(url);
        if (!resp.ok) throw new Error('YouTube API error: ' + resp.status);
        const data = await resp.json();
        if (!data.items || data.items.length === 0) {
          resultsEl.innerHTML = '<div class="small">No public live streams found for that query.</div>';
          return;
        }
        data.items.forEach(it => {
          const item = {
            videoId: it.id.videoId,
            title: it.snippet.title,
            channel: it.snippet.channelTitle,
            thumbnail: it.snippet.thumbnails.medium.url,
            publishedAt: it.snippet.publishedAt
          };
          resultsEl.appendChild(createResultItem(item));
        });
      } catch (err) {
        console.error(err);
        resultsEl.innerHTML = '<div class="small">Error searching YouTube. Check console for details.</div>';
      }
    }

    function playYouTube(videoId) {
      placeholder && placeholder.remove();
      playerContainer.innerHTML = `
        <div class="small" style="margin-bottom:8px;">Now playing (YouTube)</div>
        <iframe
          class="player"
          src="https://www.youtube.com/embed/${videoId}?autoplay=1&rel=0"
          frameborder="0" allow="autoplay; encrypted-media; picture-in-picture" allowfullscreen>
        </iframe>
      `;
    }

    function playTwitchChannel(channelName) {
      // Note: Twitch embed requires parent param matching your host (e.g., localhost)
      // and some browsers restrict autoplay. Make sure to host on your domain and add it to allowed parents.
      placeholder && placeholder.remove();
      const parentHost = window.location.hostname || 'localhost';
      playerContainer.innerHTML = `
        <div class="small" style="margin-bottom:8px;">Now playing (Twitch) — ${channelName}</div>
        <iframe
          src="https://player.twitch.tv/?channel=${encodeURIComponent(channelName)}&parent=${encodeURIComponent(parentHost)}&autoplay=true"
          class="player"
          frameborder="0"
          allowfullscreen>
        </iframe>
      `;
    }

    // allow user to paste any YouTube/Twitch/iframe link as official source
    addBtn.addEventListener('click', () => {
      const url = (manualUrl.value || '').trim();
      if (!url) return alert('Paste an official stream URL first.');
      // Simple detection
      if (url.includes('youtube.com') || url.includes('youtu.be')) {
        // extract video id if possible
        const vid = (url.match(/(?:v=|\/embed\/|youtu\.be\/)([A-Za-z0-9_-]{8,})/) || [])[1];
        if (vid) {
          resultsEl.prepend(createResultItem({ videoId: vid, title: 'Official added: ' + vid, channel: 'Manual', thumbnail: 'https://img.youtube.com/vi/' + vid + '/mqdefault.jpg', publishedAt: new Date().toISOString() }));
        } else {
          alert('Added as link — click it will open in new tab');
          const a = document.createElement('a');
          a.href = url;
          a.textContent = url;
          a.target = '_blank';
          const wrap = document.createElement('div');
          wrap.className = 'resultItem';
          wrap.appendChild(a);
          resultsEl.prepend(wrap);
        }
      } else if (url.includes('twitch.tv')) {
        const parts = url.split('/');
        const channel = parts[parts.length - 1] || parts[parts.length - 2];
        resultsEl.prepend(createResultItem({ videoId: '', title: 'Twitch: ' + channel, channel: 'Manual', thumbnail: '', publishedAt: new Date().toISOString(), twitchChannel: channel }));
        // click handler to play Twitch
        resultsEl.firstChild.addEventListener('click', () => playTwitchChannel(channel));
      } else {
        // generic entry
        const a = document.createElement('a');
        a.href = url;
        a.textContent = url;
        a.target = '_blank';
        const wrap = document.createElement('div');
        wrap.className = 'resultItem';
        wrap.appendChild(a);
        resultsEl.prepend(wrap);
      }
      manualUrl.value = '';
    });

    // search button
    searchBtn.addEventListener('click', () => searchYouTubeLive(qEl.value));

    // Quick enter
    qEl.addEventListener('keydown', (e) => { if (e.key === 'Enter') searchYouTubeLive(qEl.value); });

    // initial quick search
    // (Uncomment the next line to auto-search on load)
    // searchYouTubeLive(qEl.value);
  </script>
</body>
</html># Our-live-football-
Stream live football watch for free
