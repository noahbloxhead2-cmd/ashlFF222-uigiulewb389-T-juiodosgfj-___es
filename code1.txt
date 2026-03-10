<!doctype html>
<html>
  <head>
    <title>Forum Comments</title>
    <style>
      body { font-family: sans-serif; }
      #comments { margin-top: 20px; }
      .comment {
        padding: 8px;
        background: #eee;
        margin-bottom: 6px;
        border-radius: 4px;
      }
      #outerComments {
        overflow: scroll;
        height: 300px;
        width: 100%;
      }
      textarea {
        width: 300px;
        height: 30px;
        border: 2px solid black;
        border-radius: 10px;
        resize: none;
        overflow: hidden;
        padding: 5px;
      }
    </style>
  </head>

  <body>
    <h2>Comments</h2>
    <div id="outerComments">
      <p id="comments">Loading...</p>
    </div>

    <textarea id="username" placeholder="Your name"></textarea><br />
    <textarea id="newcomment" placeholder="Say something"></textarea><br />
    <button id="postBtn">Post Comment</button>

    <p>Usernames might not be that person — be safe!</p>

    <script type="module">
      import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
      import {
        getDatabase,
        ref,
        push,
        onValue,
        remove
      } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

      // Firebase config
      const firebaseConfig = {
        apiKey: "AIzaSyDx_P6m-wW68dfFfH3Yn8S6H3lXlNDg62g",
        authDomain: "project1-bad96.firebaseapp.com",
        databaseURL: "https://project1-bad96-default-rtdb.firebaseio.com",
        projectId: "project1-bad96",
        storageBucket: "project1-bad96.firebasestorage.app",
        messagingSenderId: "1061798946964",
        appId: "1:1061798946964:web:6d5e24253b4146af522c81",
        measurementId: "G-E6S5VB594P"
      };

      const app = initializeApp(firebaseConfig);
      const db = getDatabase(app);

      // CHANGE THIS PER PAGE
      const forumName = "IN CODE TEST";

      const commentsDiv = document.getElementById("comments");
      const newComment = document.getElementById("newcomment");
      const postBtn = document.getElementById("postBtn");
      const usernameInput = document.getElementById("username");

      // -------------------------------
      // 🔥 LOAD BANNED WORDS ONCE
      // -------------------------------
      let bannedwords = [];
      let bannedwordsLoaded = false;

      fetch("https://raw.githubusercontent.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words/master/en")
        .then(r => r.text())
        .then(t => {
          bannedwords = t
            .split(/\r?\n/)
            .map(w => w.trim().toLowerCase())
            .filter(w => w.length);

          bannedwordsLoaded = true;
          console.log("Loaded banned words:", bannedwords.length);
        })
        .catch(err => console.error("Failed to load banned words:", err));

      const allowedwords = ["bypass", "grape"];

      function isBadButNotAllowed(text) {
        if (!bannedwordsLoaded) return false;

        const lower = text.toLowerCase();

        for (const bad of bannedwords) {
          if (lower.includes(bad)) {
            for (const allow of allowedwords) {
              if (lower.includes(allow.toLowerCase())) {
                return false;
              }
            }
            return true;
          }
        }
        return false;
      }

      // -------------------------------
      // 🔥 LOAD COMMENTS
      // -------------------------------
      function loadComments() {
        const commentsRef = ref(db, "forums/" + forumName + "/comments");

        onValue(commentsRef, snapshot => {
          if (!snapshot.exists()) {
            commentsDiv.innerHTML = "No comments yet...";
            return;
          }

          let items = [];
          snapshot.forEach(child => items.push(child.val()));
          items.reverse();

          let html = "";
          items.forEach(data => {
            html += `<div class="comment"><b>${data.user}</b>: ${data.text}</div>`;
          });

          commentsDiv.innerHTML = html;
        });
      }

      function deleteCommentsInForum() {
        const commentsRef = ref(db, "forums/" + forumName + "/comments");
        remove(commentsRef);
      }

      // -------------------------------
      // 🔥 POST COMMENT
      // -------------------------------
      postBtn.onclick = () => {
        let text = newComment.value.trim();
        let username = usernameInput.value.trim() || "Anonymous";

        if (!text) return;

        if (text === "/clearchat") {
          deleteCommentsInForum();
          return;
        }

        if (isBadButNotAllowed(text)) {
          text = "This message had a swear word";
        }

        if (isBadButNotAllowed(username)) {
          username = "Person who swears";
        }

        const commentsRef = ref(db, "forums/" + forumName + "/comments");

        push(commentsRef, {
          user: username,
          text: text,
          time: Date.now()
        });

        newComment.value = "";
      };

      loadComments();
    </script>
  </body>
</html>
