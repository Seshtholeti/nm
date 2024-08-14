.App-logo {
  height: 40vmin;
  pointer-events: none;
}

@media (prefers-reduced-motion: no-preference) {
  .App-logo {
    animation: App-logo-spin infinite 20s linear;
  }
}

.App-header {
  background-color: #282c34;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: calc(10px + 2vmin);
  color: white;
  width: 100vw;
}

.App-link {
  color: #61dafb;
}

@keyframes App-logo-spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* App.css */
@keyframes marquee {
  0% {
    transform: translateX(100%);
  }
  100% {
    transform: translateX(-100%);
  }
}
.marquee {
  overflow: hidden;
  white-space: nowrap;
  box-sizing: border-box;
}
.marquee span {
  display: inline-block;
  padding-left: 100%;
  animation: marquee 10s linear infinite;
}

.App {
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 95vh;
  background-color: #f0f2f5;
  text-align: center;
  width: 100%;
  /* margin-left: 6px; */
}
.welcome-container {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.welcome-container h1 {
  margin-bottom: 1rem;
  font-size: 2rem;
  color: #333;
}
.welcome-container p {
  margin-bottom: 1.5rem;
  font-size: 1.2rem;
  color: #666;
}
.sign-in-button {
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  color: white;
  background-color: #0078d4;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.sign-in-button:hover {
  background-color: #005a9e;
}
