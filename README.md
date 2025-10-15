<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>LK Streaming</title>
  <style>
    body { font-family: Arial, sans-serif; background: #111; color: white; margin: 0; padding: 20px; text-align: center; }
    header { background: #222; padding: 15px; text-align: center; font-size: 32px; font-weight: bold; cursor: pointer; color: #1e90ff; }
    header:hover { color: #00ff99; }

    .input-box { padding: 12px; border-radius: 5px; border: 3px solid #ff4500; background: #111; color: white; text-align: center; margin: 10px 5px; }

    button { background: crimson; color: white; border: none; border-radius: 5px; cursor: pointer; padding: 10px 16px; }
    .video-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(360px, 1fr)); gap: 25px; margin-top: 20px; justify-items: center; }
    .video-card { background: #222; padding: 15px; border-radius: 12px; text-align: center; position: relative; }
    video { width: 100%; border-radius: 12px; height: auto; }
    .hidden { display: none; }
    .delete-btn { background: darkred; position: absolute; top: 10px; right: 10px; padding: 5px 10px; font-size: 12px; }
    #logoutBtn { background: darkorange; margin-top: 10px; }
    .show-hide { cursor: pointer; color: #fff; font-weight: bold; }
    .no-videos { text-align: center; font-size: 18px; color: #ccc; margin-top: 20px; }

    /* Movie search box styling */
    #searchInput { width: 600px; padding: 12px; border-radius: 5px; border: 3px solid #32cd32; margin-top: 20px; text-align: center; background: #111; color: white; }

    #adminPanel { margin-top: 20px; }
  </style>
</head>
<body>
  <header onclick="toggleLogin()">🎬 STREAM HUB</header>

  <!-- Hidden Admin Login -->
  <div id="loginSection" class="hidden">
    <input type="password" id="adminPassword" placeholder="Enter Admin Password" class="input-box">
    <span class="show-hide" onclick="toggleLoginPassword()">👁️</span>
    <button onclick="checkLogin()">Login</button>
  </div>

  <!-- Admin Panel -->
  <div id="adminPanel" class="hidden">
    <input type="file" id="uploadVideo" accept="video/*"><br>
    <input type="text" id="videoTitle" placeholder="Movie Title" class="input-box"><br>
    <button onclick="postVideo()">Post Video</button>
    <button id="logoutBtn" onclick="logout()">Logout</button>
    <hr>
  </div>

  <!-- Visitor Homepage -->
  <input type="text" id="searchInput" placeholder="Search movies..." onkeyup="searchVideos()">
  <h2>🎞️ Latest Movies</h2>
  <div class="video-list" id="videoList"></div>
  <div id="noVideos" class="no-videos">No videos uploaded yet</div>

  <script>
    let storedPassword = "timepass"; // ✅ Updated password
    let isAdmin = false;
    let videos = JSON.parse(localStorage.getItem("kushVideos")) || [];

    function toggleLoginPassword() {
      const input = document.getElementById("adminPassword");
      input.type = input.type === "password" ? "text" : "password";
    }

    function toggleLogin() {
      const loginBox = document.getElementById("loginSection");
      loginBox.classList.toggle("hidden");
    }

    function renderVideos(list = videos) {
      const videoList = document.getElementById("videoList");
      const noVideos = document.getElementById("noVideos");
      videoList.innerHTML = "";

      if (list.length === 0) {
        noVideos.style.display = "block";
      } else {
        noVideos.style.display = "none";
        list.forEach((v, index) => {
          const card = document.createElement("div");
          card.className = "video-card";
          let deleteBtn = "";
          if (isAdmin) deleteBtn = `<button class="delete-btn" onclick="deleteVideo(${index})">Delete</button>`;
          card.innerHTML = `
            <h3>${v.title}</h3>
            <video controls>
              <source src="${v.src}" type="video/mp4">
              Your browser does not support video.
            </video>
            ${deleteBtn}
          `;
          videoList.appendChild(card);
        });
      }
    }

    // 🔍 Flexible search (matches even a single letter, ignore case)
    function searchVideos() {
      const query = document.getElementById("searchInput").value.toLowerCase().trim();
      if (query === "") {
        renderVideos(videos);
        return;
      }

      const filtered = videos.filter(v => v.title.toLowerCase().includes(query));
      renderVideos(filtered);
    }

    function postVideo() {
      const fileInput = document.getElementById("uploadVideo");
      const titleInput = document.getElementById("videoTitle");
      if (fileInput.files.length === 0 || titleInput.value === "") return;

      const file = fileInput.files[0];
      const reader = new FileReader();

      reader.onload = function(e) {
        const videoData = { title: titleInput.value, src: e.target.result };
        videos.unshift(videoData);
        localStorage.setItem("kushVideos", JSON.stringify(videos));
        renderVideos();
        fileInput.value = "";
        titleInput.value = "";
      };

      reader.readAsDataURL(file);
    }

    function deleteVideo(index) {
      if (!isAdmin) return;
      videos.splice(index, 1);
      localStorage.setItem("kushVideos", JSON.stringify(videos));
      renderVideos();
    }

    function checkLogin() {
      const password = document.getElementById("adminPassword").value;
      if (password === storedPassword) {
        isAdmin = true;
        document.getElementById("adminPanel").classList.remove("hidden");
        document.getElementById("loginSection").classList.add("hidden");
        renderVideos();
      } else {
        alert("Wrong password!");
      }
    }

    function logout() {
      isAdmin = false;
      document.getElementById("adminPanel").classList.add("hidden");
      document.getElementById("loginSection").classList.add("hidden");
      document.getElementById("adminPassword").value = "";
      renderVideos();
    }

    window.onbeforeunload = function() {
      logout();
    };

    renderVideos();
  </script>
</body>
</html>
