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
this is my app.js

import React, { useState, useEffect } from "react";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faSignOutAlt } from "@fortawesome/free-solid-svg-icons";
function Header({ onLogout }) {
  const [currentTime, setCurrentTime] = useState(new Date());
  const [isHovered, setIsHovered] = useState(false);
  useEffect(() => {
    const timer = setInterval(() => {
      setCurrentTime(new Date());
    }, 1000);
    return () => clearInterval(timer);
  }, []);
  const headerStyle = {
    color: "white",
    backgroundColor: "#820882",
    padding: "10px",
    height: "30px",
    display: "flex",
    alignItems: "center",
    fontSize: "40px",
    width: "100vw",
    marginTop: "50px",
    position: "fixed",
    top: "-50px",
    justifyContent: "space-between",
    // position: "relative",
    // bottom: "28px",
  };
  const headerTitle = {
    paddingBottom: "5px",
    marginLeft: "60px",
  };
  const formatDate = (date) => {
    const options = { timeZone: "Europe/Berlin" };
    const day = String(
      date.toLocaleDateString("en-GB", options).split("/")[0]
    ).padStart(2, "0");
    const month = String(
      date.toLocaleDateString("en-GB", options).split("/")[1]
    ).padStart(2, "0");
    const year = date.toLocaleDateString("en-GB", options).split("/")[2];
    return `${day}.${month}.${year}`;
  };
  const formatTime = (date) => {
    const options = {
      hour: "2-digit",
      minute: "2-digit",
      second: "2-digit",
      hour12: false,
      timeZone: "Europe/Berlin",
    };
    return date.toLocaleTimeString("en-GB", options);
  };
  const iconContainerStyle = {
    display: "flex",
    alignItems: "center",
    marginRight: "60px",
  };
  const iconStyle = {
    marginLeft: "10px",
    fontSize: "30px",
    color: "white",
    cursor: "pointer",
  };
  const iconHoverStyle = {
    color: "#c9302c",
  };
  const logoutTextStyle = {
    fontSize: "10px",
    marginLeft: "15px",
    visibility: isHovered ? "visible" : "hidden",
  };
  return (
    <div style={headerStyle}>
      <div style={headerTitle}>Anrufstatistik</div>
      <div style={iconContainerStyle}>
        <div>
          {formatDate(currentTime)}&nbsp;&nbsp;{formatTime(currentTime)}
        </div>
        <div
          style={logoutTextStyle}
          onMouseOver={() => setIsHovered(true)}
          onMouseOut={() => setIsHovered(false)}
          onClick={onLogout}
        >
          Logout
        </div>
        <FontAwesomeIcon
          icon={faSignOutAlt}
          style={iconStyle}
          onMouseOver={(e) => {
            e.currentTarget.style.color = iconHoverStyle.color;
            setIsHovered(true);
          }}
          onMouseOut={(e) => {
            e.currentTarget.style.color = iconStyle.color;
            setIsHovered(false);
          }}
          onClick={onLogout}
        />
      </div>
    </div>
  );
}
export default Header;
this is my header.js

import React from "react";
import "./App.css";
function ReservationCenterHeader({ avg }) {
  console.log(avg);
  const footerStyle = {
    color: "white",
    // Dark blue color
    backgroundColor: "#820882",
    // Adjust font size as needed
    paddingLeft: "500px",
    // height: "0.7vh",
    width: "100vw",
    display: "flex",
    alignItems: "center",
    fontSize: "38px",
    // marginTop: "90px",
    position: "fixed",
    bottom: "-36px",
    // textAlign: "centre",
    fontWeight: "bold",
    // top: "32px",
  };

  // const Marquee = ({ text }) => {
  return (
    <div className="marquee">
      <p style={footerStyle}>Total Angenommen: {avg}</p>
    </div>
  );
  // };
}
export default ReservationCenterHeader;
this is my footer.js
