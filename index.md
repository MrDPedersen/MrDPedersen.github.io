<!DOCTYPE html>
<html>
  <head>
    <!-- // This stylesheet is completely optional, it's just the first one I found to make the page a b-->
    <link
      rel="stylesheet"
      href="https://unpkg.com/purecss@2.0.5/build/pure-min.css"
      integrity="sha384-LTIDeidl25h2dPxrB2Ekgc9c7sEC3CWGM6HeFmuDNUjX76Ert4Z4IY714dhZHPLd"
      crossorigin="anonymous"
    />
    <script>
      // WebXR requires HTTPS, so the site doesn't work if someone manually enters
      // the URL and ends up using HTTP. To work around this, force redirect from
      // http to https for non-localhost addresses.
      if (
        window.location.protocol == "http:" &&
        window.location.hostname != "localhost" &&
        window.location.hostname != "127.0.0.1" &&
        window.location.hostname != "[::1]"
      ) {
        window.location = window.location.href.replace("http:", "https:");
      }
    </script>
  </head>
  <body>
    <div class="content" style="margin-left: 20px; margin-right: 20px;">
      <div style="width: 100%; text-align: center;">
        <div id="player"></div>
      </div>
      <h1>Oculus Quest background youtube player</h1>
      <h2>How to use</h2>
      <ul>
        <li>
          Go to <br />

          <b>https://oqbackg.glitch.me</b>
          <br />
          <br />
          <b>https://vrworkout.at/music.html?v=your_video_id</b>
          <br />&nbsp;
        </li>
        <li>
          Switch to desktop mode <br />
          <img
            src="https://cdn.glitch.com/475fdad1-f79e-4a77-880e-efdac6d6c308%2Fenable_desktop_mode.jpg?v=1615308433790"
            style="width: 480px; height: auto; display: block;"
            alt="enable desktop mode"
          />
        </li>
        <li>Press play and then start the game you want to play</li>
        <li>To stop you have to go back to this window and close the tab</li>
      </ul>

      <h2>
        How it works?
      </h2>
      <p>
        Check the source of the page, it's very simple. We keep calling a
        function that checks if the player has been paused, which happens
        automatically if the tab goes inactive.
      </p>
      <p>
        If it has been paused we start playing it again.
      </p>
    </div>
    <script>
      // 2. This code loads the IFrame Player API code asynchronously.
      var tag = document.createElement("script");

      tag.src = "https://www.youtube.com/iframe_api";
      var firstScriptTag = document.getElementsByTagName("script")[0];
      firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

      // 3. This function creates an <iframe> (and YouTube player)
      //    after the API code downloads.
      var player;

      function onYouTubeIframeAPIReady() {
        console.log(`API ready connect to server`);
        var parameters = {};
        if (window.location.hash.indexOf("#") == 0) {
          parameters = new URLSearchParams(window.location.hash.substr(1));
        } else {
          parameters = new URLSearchParams(window.location.search);
        }

        var video_id = "o-Lpq1pjQcc";
        var loop = 1;
        if (parameters.has("v")) {
          video_id = parameters.get("v");
        }

        //Loop does not work yet
        if (parameters.has("loop")) {
          loop = parameters.get("loop");
        }

        playVideo(video_id, loop);
        // var socket = new WebSocket("ws://192.168.1.61:19338");
        // socket.onopen = onSocketOpen;
        // socket.onmessage = onSocketMessage;
        // socket.onclose = onSocketClose;
        // socket.onerror = onSocketError;
      }

      // 4. The API will call this function when the video player is ready.

      var event_player;
      function onPlayerReady(event) {
        event_player = event.target;
        //setTimeout(startVideo,2000);
      }

      function startVideo() {
        //event_player.mute();
        event_player.playVideo();
      }

      var persistent_play_active = false;
      function keepPlaying() {
        var state = player.getPlayerState();
        if (state == 2) {
          console.log(`Video is paused. Try to restart`);
          event_player.playVideo();
        } else {
          console.log(`Current player state: ${state}`);
        }
        if (persistent_play_active) {
          setTimeout(keepPlaying, 1000);
        }
      }

      function playVideo(video_id, loop) {
        console.log(`Loop: ${loop}`);

        player = new YT.Player("player", {
          height: "390",
          width: "640",
          videoId: video_id,
          loop: loop,
          origin: "https://vrworkout.at/",
          events: {
            onReady: onPlayerReady,
            onStateChange: onPlayerStateChange
          }
        });
      }

      var done = false;
      function onPlayerStateChange(event) {
        if (event.data == YT.PlayerState.PLAYING) {
          if (!persistent_play_active) {
            persistent_play_active = true;
            keepPlaying();
          }
        }
        //console.log(JSON.stringify(event.data));
      }

      function stopVideo() {
        player.stopVideo();
      }

      function onSocketOpen(e) {
        console.log("[open] Connection established");
        //alert("Sending to server");
        //socket.send("My name is John");
      }

      function onSocketMessage(event) {
        console.log(`[message] Data received from server: ${event.data}`);

        var msg = event.data;
        var command = JSON.parse(msg);
        if (command["cmd"] == "play") {
          console.log(`Playing video: ${command["video"]}`);
          playVideo(command["video"]);
        } else {
          console.log(`Unknown command`);
        }
      }

      function onSocketClose(event) {
        if (event.wasClean) {
          console.log(
            `[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`
          );
        } else {
          // e.g. server process killed or network down
          // event.code is usually 1006 in this case
          console.log("[close] Connection died");
        }
      }

      function onSocketError(error) {
        console.log(`[error] ${error.message}`);
      }
    </script>
  </body>
</html>
