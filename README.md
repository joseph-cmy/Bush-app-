<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BUSH 🇺🇬 ♻️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background-color: #111; color: white; overflow-y: auto; }
        .top-nav, .search-bar, .category-nav { position: fixed; width: 100%; background-color: #111; z-index: 1000; }
        .top-nav { display: flex; justify-content: space-between; align-items: center; padding: 10px 20px; top: 0; }
        .search-bar { top: 50px; padding: 10px; }
        .search-bar input { width: 100%; padding: 10px; border: none; background-color: #333; color: white; border-radius: 5px; }
        .category-nav { top: 100px; display: flex; overflow-x: auto; padding: 10px; white-space: nowrap; }
        .category-item { padding: 10px 15px; cursor: pointer; text-transform: uppercase; font-size: 14px; font-weight: bold; color: white; }
        .category-item.active { background-color: #333; border-radius: 5px; }
        .video-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; padding: 20px; margin-top: 140px; }
        .video-item { background-color: white; border-radius: 8px; overflow: hidden; cursor: pointer; transition: 0.3s ease; position: relative; }
        .video-item:hover { transform: scale(1.03); }
        .video-item img { width: 100%; height: 150px; object-fit: cover; display: block; }
        .video-item video, .video-item iframe { width: 100%; height: 150px; object-fit: cover; display: none; }
        .video-item .info { padding: 10px; font-size: 12px; color: black; }
        .fullscreen-video { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: black; z-index: 9999; display: none; justify-content: center; align-items: center; flex-direction: column; }
        .fullscreen-video video { width: 80%; height: 80%; object-fit: cover; }
        .fullscreen-video iframe { width: 80%; height: 80%; }
        .fullscreen-video .close-btn { position: absolute; top: 20px; right: 20px; background-color: rgba(0, 0, 0, 0.5); color: white; padding: 10px; cursor: pointer; }
        .video-controls { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); display: flex; justify-content: space-between; width: 80%; }
        .control-btn { background-color: rgba(0, 0, 0, 0.7); color: white; padding: 10px; cursor: pointer; border: none; border-radius: 5px; font-size: 18px; }
    </style>
</head>
<body>
    <div class="top-nav">
        <div class="left"><i class="fa fa-bell"></i> <i class="fa fa-search"></i></div>
        <div class="right"><span>BUSH 🇺🇬 ♻️</span></div>
    </div>
    <div class="search-bar"><input type="text" id="search-input" placeholder="Search videos..."></div>
    <div class="category-nav">
        <div class="category-item active">All</div>
        <div class="category-item">Travel</div>
        <div class="category-item">Sports</div>
        <div class="category-item">Culture</div>
        <div class="category-item">Music</div>
        <div class="category-item">Politics</div>
    </div>
    <div class="video-grid" id="video-grid"></div><!-- Fullscreen Video Container -->
<div class="fullscreen-video" id="fullscreen-video">
    <div class="close-btn" id="close-btn">X</div>
    <video id="fullscreen-video-player" preload="auto" controls></video>
    <iframe id="fullscreen-video-iframe" frameborder="0" allowfullscreen></iframe>
    <div class="video-controls">
        <button class="control-btn" id="rewind-btn">Rewind</button>
        <button class="control-btn" id="play-pause-btn">Play</button>
        <button class="control-btn" id="forward-btn">Forward</button>
    </div>
</div>

<script>
    const PEXELS_API_KEY = "E3tvYZ7r5rcJ87eypOMBgGkQZb9Ksc1dniux3X9hVpgOMAsGEN2XIok8";
    const YOUTUBE_API_KEY = "AIzaSyDxZ6KqzYAcfzeEYcp4_y1Ijn3eeUCh5Ps";
    const MIXKIT_API_URL = "https://api.mixkit.co/videos/search/";
    const ARCHIVE_API_URL = "https://archive.org/advancedsearch.php?q=media_type%3Avideo&output=json";

    let currentPage = 1, isLoading = false;
    let fetchedVideos = { youtube: [], pexels: [], mixkit: [], archive: [] };

    // Fetch and display videos
    async function fetchVideos() {
        if (isLoading) return;
        isLoading = true;

        // Fetch videos in the desired order
        await fetchMixkitVideos();
        await fetchArchiveVideos();
        await fetchYouTubeVideos();
        await fetchPexelsVideos();

        isLoading = false;
    }

    // Fetch Mixkit videos
    async function fetchMixkitVideos() {
        const url = `${MIXKIT_API_URL}?page=${currentPage}&limit=50`;
        try {
            const response = await fetch(url);
            const data = await response.json();
            const newVideos = data.results.map(video => ({
                type: "mixkit",
                title: video.title,
                thumbnail: video.preview_url,
                src: video.url
            }));
            addUniqueVideos(newVideos, 'mixkit');
        } catch (error) {
            console.error("Error fetching Mixkit videos:", error);
        }
    }

    // Fetch Archive.org videos
    async function fetchArchiveVideos() {
        const url = `${ARCHIVE_API_URL}&rows=50&page=${currentPage}`;
        try {
            const response = await fetch(url);
            const data = await response.json();
            const newVideos = data.response.docs.map(video => ({
                type: "archive",
                title: video.title,
                thumbnail: video.thumbnail || 'default-thumbnail.jpg',
                src: `https://archive.org/download/${video.identifier}/${video.identifier}.mp4`
            }));
            addUniqueVideos(newVideos, 'archive');
        } catch (error) {
            console.error("Error fetching Archive.org videos:", error);
        }
    }

    // Fetch YouTube videos
    async function fetchYouTubeVideos() {
        const url = `https://www.googleapis.com/youtube/v3/videos?part=snippet&chart=mostPopular&maxResults=50&regionCode=UG&key=${YOUTUBE_API_KEY}`;
        try {
            const response = await fetch(url);
            const data = await response.json();
            const newVideos = data.items.map(video => ({
                type: "youtube",
                title: video.snippet.title,
                thumbnail: video.snippet.thumbnails.medium.url,
                src: `https://www.youtube.com/embed/${video.id}?autoplay=1&muted=1`
            }));
            addUniqueVideos(newVideos, 'youtube');
        } catch (error) {
            console.error("Error fetching YouTube videos:", error);
        }
    }

    // Fetch Pexels videos
    async function fetchPexelsVideos() {
        const url = `https://api.pexels.com/videos/popular?page=${currentPage}&per_page=50`;
        try {
            const response = await fetch(url, { headers: { Authorization: PEXELS_API_KEY } });
            const data = await response.json();
            const newVideos = data.videos.map(video => ({
                type: "pexels",
                title: "Pexels Video",
                thumbnail: video.image,
                src: video.video_files[0].link
            }));
            addUniqueVideos(newVideos, 'pexels');
        } catch (error) {
            console.error("Error fetching Pexels videos:", error);
        }
    }

    // Function to add unique videos to the list and display them
    function addUniqueVideos(newVideos, source) {
        newVideos.forEach(video => {
            if (!fetchedVideos[source].find(v => v.src === video.src)) {
                fetchedVideos[source].push(video);
                displayVideos([video]);
            }
        });
    }

    // Function to display videos
    function displayVideos(videos) {
        const videoGrid = document.getElementById('video-grid');
        videos.forEach(video => {
            const videoItem = document.createElement('div');
            videoItem.className = 'video-item';
            videoItem.innerHTML = `
                <img src="${video.thumbnail}" loading="lazy">
                <video preload="auto" src="${video.src}" muted></video>
                <iframe src="${video.src}" frameborder="0" allowfullscreen></iframe>
                <div class="info">${video.title}</div>
            `;
            videoGrid.appendChild(videoItem);

            // Handle click to open video in fullscreen
            videoItem.addEventListener('click', () => {
                const fullscreenVideo = document.getElementById('fullscreen-video');
                const videoPlayer = document.getElementById('fullscreen-video-player');
                const iframePlayer = document.getElementById('fullscreen-video-iframe');
                const playPauseBtn = document.getElementById('play-pause-btn');
                const rewindBtn = document.getElementById('rewind-btn');
                const forwardBtn = document.getElementById('forward-btn');
                const closeBtn = document.getElementById('close-btn');

                fullscreenVideo.style.display = 'flex';

                // Hide iframe and show video player based on video type
                if (video.type === "youtube") {
                    iframePlayer.style.display = 'block';
                    videoPlayer.style.display = 'none';
                    iframePlayer.src = video.src;
                } else {
                    iframePlayer.style.display = 'none';
                    videoPlayer.style.display = 'block';
                    videoPlayer.src = video.src;
                    videoPlayer.play();
                }

                // Play/Pause functionality
                playPauseBtn.addEventListener('click', () => {
                    if (videoPlayer.paused) {
                        videoPlayer.play();
                        playPauseBtn.innerText = "Pause";
                    } else {
                        videoPlayer.pause();
                        playPauseBtn.innerText = "Play";
                    }
                });

                // Rewind functionality
                rewindBtn.addEventListener('click', () => {
                    videoPlayer.currentTime -= 10;
                });

                // Forward functionality
                forwardBtn.addEventListener('click', () => {
                    videoPlayer.currentTime += 10;
                });

                // Close the fullscreen video
                closeBtn.addEventListener('click', () => {
                    fullscreenVideo.style.display = 'none';
                    videoPlayer.src = '';
                    iframePlayer.src = '';
                });
            });
        });
    }

    // Handle Scroll-Down for Infinite Scroll
    window.addEventListener('scroll', () => {
        const bottom = document.documentElement.scrollHeight === document.documentElement.scrollTop + window.innerHeight;
        if (bottom) {
            currentPage++;
            fetchVideos();
        }
    });

    // Initialize the app by fetching videos
    fetchVideos();
</script>

</body>
</html>
