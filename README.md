import React from "react";
import "./App.css";
import Dashboard from "./Dashboard";
import Header from "./Header";
import Footer from "./Footer";
function App() {
  return (
    <div className="App">
      <header>{/* <h1>QuickSight Dashboard</h1> */}</header>
      <main>
        <Header />
        <Dashboard />
        <Footer />
      </main>
    </div>
  );
}
export default App;

this my refernce app.js

import React, { useEffect, useState } from "react";
import {
  AuthenticatedTemplate,
  UnauthenticatedTemplate,
  useMsal,
  MsalProvider,
} from "@azure/msal-react";
import { loginRequest } from "./authConfig";
const WrappedView = () => {
  const { instance, accounts } = useMsal(); // Get instance and accounts from useMsal hook
  const [accessToken, setAccessToken] = useState(null);
  useEffect(() => {
    const account = accounts[0]; // Get the first account from the accounts array
    if (account) {
      instance
        .acquireTokenSilent({
          ...loginRequest,
          account: account,
        })
        .then((response) => {
          setAccessToken(response.accessToken);
        })
        .catch((error) => {
          console.error("Error acquiring token silently:", error);
        });
    }
  }, [instance, accounts]);
  const handleLoginPopup = () => {
    console.log("Initiating login popup...");
    instance
      .loginPopup(loginRequest)
      .then((response) => {
        console.log("Login successful. Access token:", response.accessToken);
        setAccessToken(response.accessToken); // Update accessToken state
      })
      .catch((error) => {
        console.error("Login Popup Error:", error);
      });
  };
  return (
    <div className="App">
      <AuthenticatedTemplate>
        {accessToken ? (
          <p>Authenticated Successfully with Access Token: {accessToken}</p>
        ) : (
          <p>Loading...</p>
        )}
      </AuthenticatedTemplate>
      <UnauthenticatedTemplate>
        <button onClick={handleLoginPopup}>Sign in</button>
      </UnauthenticatedTemplate>
    </div>
  );
};
const App = ({ instance }) => {
  return (
    <MsalProvider instance={instance}>
      <WrappedView />
    </MsalProvider>
  );
};
export default App;

this is my original app.js


modify my original app.js with the refernce part after successful authenticated we need to show the header, dashboard, footer components
