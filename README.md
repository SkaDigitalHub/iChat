<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    /* General Styles */
    body {
      font-family: 'Arial', sans-serif;
      background: linear-gradient(135deg, #1e3c72, #2a5298);
      color: #fff;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden;
    }

    /* Login Container */
    #login {
      background: rgba(255, 255, 255, 0.1);
      padding: 2rem;
      border-radius: 15px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
      backdrop-filter: blur(10px);
      text-align: center;
      width: 300px;
      animation: fadeIn 1s ease-in-out;
    }

    #login h2 {
      margin-bottom: 1.5rem;
      font-size: 1.8rem;
    }

    #login input {
      width: 100%;
      padding: 0.8rem;
      margin: 0.5rem 0;
      border: none;
      border-radius: 5px;
      background: rgba(255, 255, 255, 0.2);
      color: #fff;
      font-size: 1rem;
    }

    #login input::placeholder {
      color: rgba(255, 255, 255, 0.7);
    }

    #login button {
      width: 100%;
      padding: 0.8rem;
      margin: 0.5rem 0;
      border: none;
      border-radius: 5px;
      background: #ff6f61;
      color: #fff;
      font-size: 1rem;
      cursor: pointer;
      transition: background 0.3s ease;
    }

    #login button:hover {
      background: #ff3b2f;
    }

    /* Chat Container */
    #chat {
      display: none;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 15px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
      backdrop-filter: blur(10px);
      width: 400px;
      height: 600px;
      padding: 1rem;
      animation: slideIn 0.5s ease-in-out;
    }

    #messages {
      height: 400px;
      overflow-y: scroll;
      padding: 1rem;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 10px;
      margin-bottom: 1rem;
    }

    #messages p {
      background: rgba(255, 255, 255, 0.2);
      padding: 0.8rem;
      border-radius: 10px;
      margin: 0.5rem 0;
      animation: fadeInMessage 0.5s ease-in-out;
    }

    #message, #receiver {
      width: 100%;
      padding: 0.8rem;
      margin: 0.5rem 0;
      border: none;
      border-radius: 5px;
      background: rgba(255, 255, 255, 0.2);
      color: #fff;
      font-size: 1rem;
    }

    #chat button {
      width: 100%;
      padding: 0.8rem;
      margin: 0.5rem 0;
      border: none;
      border-radius: 5px;
      background: #ff6f61;
      color: #fff;
      font-size: 1rem;
      cursor: pointer;
      transition: background 0.3s ease;
    }

    #chat button:hover {
      background: #ff3b2f;
    }

    /* Active Users Container */
    #users {
      background: rgba(255, 255, 255, 0.1);
      border-radius: 10px;
      padding: 1rem;
      margin-top: 1rem;
    }

    #users h3 {
      margin-bottom: 1rem;
    }

    #users p {
      display: flex;
      align-items: center;
      margin: 0.5rem 0;
    }

    /* Animations */
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(-20px); }
      to { opacity: 1; transform: translateY(0); }
    }

    @keyframes slideIn {
      from { opacity: 0; transform: translateX(-20px); }
      to { opacity: 1; transform: translateX(0); }
    }

    @keyframes fadeInMessage {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }
  </style>
</head>
<body>
  <div id="login">
    <h2>Login</h2>
    <input type="text" id="username" placeholder="Username">
    <input type="password" id="password" placeholder="Password">
    <button onclick="login()">Login</button>
    <button onclick="signup()">Sign Up</button>
  </div>
  <div id="chat">
    <h2>Chat</h2>
    <div id="messages"></div>
    <input type="text" id="message" placeholder="Type a message">
    <input type="text" id="receiver" placeholder="Enter receiver's username (or ALL for group chat)">
    <button onclick="sendMessage()">Send</button>
    <button onclick="logout()">Logout</button>
  </div>
  <div id="users"></div>

  <script>
    let currentUser = null;

    async function login() {
      const username = document.getElementById("username").value;
      const password = document.getElementById("password").value;
      const response = await fetchPost({ action: "login", username, password });
      if (response.success) {
        currentUser = username;
        document.getElementById("login").style.display = "none";
        document.getElementById("chat").style.display = "block";
        loadMessages();
        loadActiveUsers();
        setInterval(loadMessages, 1000);
        setInterval(loadActiveUsers, 5000);
      } else {
        alert(response.message);
      }
    }

    async function signup() {
      const username = document.getElementById("username").value;
      const password = document.getElementById("password").value;
      const response = await fetchPost({ action: "signup", username, password });
      if (response.success) {
        alert("Signup successful! Please log in.");
      } else {
        alert(response.message);
      }
    }

    async function sendMessage() {
      const message = document.getElementById("message").value;
      const receiver = document.getElementById("receiver").value || "ALL";
      await fetchPost({ action: "sendMessage", sender: currentUser, receiver, message });
      document.getElementById("message").value = "";
      document.getElementById("receiver").value = "";
      loadMessages();
    }

    async function loadMessages() {
      const response = await fetchPost({ action: "getMessages", username: currentUser });
      const messagesDiv = document.getElementById("messages");
      messagesDiv.innerHTML = response.messages.map(
        (msg) => `<p><strong>${msg[1]}:</strong> ${msg[3]} <em>(${msg[0]})</em></p>`
      ).join("");
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }

    async function loadActiveUsers() {
      const response = await fetchPost({ action: "getActiveUsers" });
      const usersDiv = document.getElementById("users");
      usersDiv.innerHTML = "<h3>Active Users</h3>" + response.activeUsers.map(
        (user) => `<p>${user}</p>`
      ).join("");
    }

    async function logout() {
      await fetchPost({ action: "logout", username: currentUser });
      location.reload();
    }

    async function fetchPost(data) {
      const response = await fetch("https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data),
      });
      return await response.json();
    }
  </script>
</body>
</html>
