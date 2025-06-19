Enter
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Podcastolax - Podcast Player</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 2rem;
      background: #f9f9f9;
      color: #333;
    }
    h1 {
      color: #2c3e50;
    }
    input[type="text"] {
      width: 80%;
      padding: 0.5rem;
      font-size: 1rem;
    }
    button {
      padding: 0.5rem 1rem;
      font-size: 1rem;
      margin-left: 0.5rem;
      cursor: pointer;
    }
    .episode {
      background: white;
      padding: 1rem;
      margin: 1rem 0;
      border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    audio {
      width: 100%;
      margin-top: 0.5rem;
    }
    .download-link {
      display: inline-block;
      margin-top: 0.5rem;
      color: #2980b9;
      text-decoration: none;
    }
    .error {
      color: red;
      margin-top: 1rem;
    }
  </style>
</head>
<body>
  <h1>Podcastolax - Podcast Player</h1>
  <p>Paste your podcast RSS feed URL below and click "Load Episodes":</p>
  <input type="text" id="rssUrl" placeholder="Enter podcast RSS feed URL here" />
  <button onclick="loadPodcast()">Load Episodes</button>
  <div id="episodes"></div>
  <div id="error" class="error"></div>

  <script>
    async function loadPodcast() {
      const rssUrl = document.getElementById('rssUrl').value.trim();
      const episodesDiv = document.getElementById('episodes');
      const errorDiv = document.getElementById('error');
      episodesDiv.innerHTML = '';
      errorDiv.textContent = '';

      if (!rssUrl) {
        errorDiv.textContent = 'Please enter a valid RSS feed URL.';
        return;
      }

      try {
        // Use a CORS proxy because many RSS feeds block cross-origin requests
        // You can replace this with your own proxy if needed
        const proxyUrl = 'https://api.allorigins.win/get?url=' + encodeURIComponent(rssUrl);
        const response = await fetch(proxyUrl);
        if (!response.ok) throw new Error('Failed to fetch RSS feed.');

        const data = await response.json();
        const parser = new DOMParser();
        const xmlDoc = parser.parseFromString(data.contents, 'application/xml');

        const items = xmlDoc.querySelectorAll('item');
        if (items.length === 0) {
          errorDiv.textContent = 'No episodes found in this RSS feed.';
          return;
        }

        items.forEach(item => {
          const title = item.querySelector('title')?.textContent || 'No title';
          const enclosure = item.querySelector('enclosure');
          const audioUrl = enclosure?.getAttribute('url');
          const length = enclosure?.getAttribute('length');
          const duration = item.querySelector('itunes\\:duration')?.textContent || 'Unknown length';

          if (!audioUrl) return; // skip if no audio

          const episodeDiv = document.createElement('div');
          episodeDiv.className = 'episode';

          episodeDiv.innerHTML = `
            <h3>${title}</h3>
            <p>Duration: ${duration}</p>
            <audio controls preload="none" src="${audioUrl}"></audio>
            <br/>
            <a class="download-link" href="${audioUrl}" download>Download Episode</a>
          `;

          episodesDiv.appendChild(episodeDiv);
        });
      } catch (error) {
        errorDiv.textContent = 'Error loading RSS feed: ' + error.message;
      }
    }
  </script>
</body>
</html>
