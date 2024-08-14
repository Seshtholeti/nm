import React, { useEffect, useState } from "react";
import Header from "./Header";
import img1 from "./img/img1.jpg";
import img2 from "./img/img2.jpg";
const columnStyle = {
  flex: 1,
  display: "flex",
  flexDirection: "column",
  marginTop: "18px",
  marginBottom: "38px",
  marginRight: "20px",
  gap: "1px",
  justifyContent: "center",
  alignItems: "center",
};
const containerStyle = {
  display: "flex",
  alignItems: "flex-start",
  color: "#333",
  padding: "20px",
  height: "80.9vh",
  width: "100vw",
  boxSizing: "border-box",
};
const imageContainerStyle = {
  width: "80%",
  height: "90%",
  display: "flex",
  justifyContent: "center",
  alignItems: "center",
  borderRadius: "5px",
  marginLeft: "10px",
  marginTop: "25px",
};
const imageStyle = {
  width: "100%",
  height: "100%",
  borderRadius: "5px",
};
const dataContainerStyle = {
  display: "flex",
  flexDirection: "row",
  justifyContent: "space-between",
  width: "100%",
  height: "100%",
  marginLeft: "10px",
  fontSize: "40px",
  marginTop: "20px",
};
const cardStyle = {
  backgroundColor: "#820882",
  padding: "23px",
  borderRadius: "5px",
  display: "flex",
  justifyContent: "space-between",
  alignItems: "center",
  height: "30px", // Increased card height
  lineHeight: "60px", // Ensure text is vertically centered
  color: "#fff",
  fontSize: "31px", // Increased font size
  width: "90%",
  fontWeight: "700",
  transition: "background-color 0.3s ease",
  overflow: "hidden", // Prevent overflow of text
  textOverflow: "ellipsis", // Handle overflow with ellipsis
  whiteSpace: "nowrap", // Ensure text is on a single line
};
const hoveredCardStyle = {
  backgroundColor: "red",
  color: "white",
  cursor: "pointer",
};
const keyTranslations = {
  DEPARTMENT: "Leitung",
  CIQ: "Warteschleife",
  LWT: "Längste Wartezeit",
  LOST_CALLS: "Verpasst",
  OFFERED: "Gesamt",
  ANS: "Angenommen",
  ANS_RATE: "Angenommen %",
  RDY: "Bereit",
  TALK: "Im Gespräch",
  NOT_RDY: "Nicht bereit",
  ONLINE: "Angemeldet",
};
const orderedKeys = [
  "DEPARTMENT",
  "OFFERED",
  "ANS",
  "LOST_CALLS",
  "ANS_RATE",
  "CIQ",
  "LWT",
  "ONLINE",
  "RDY",
  "TALK",
  "NOT_RDY",
];
const App = (props) => {
  const [data, setData] = useState([]);
  const [currentImageIndex, setCurrentImageIndex] = useState(0);
  const [hoveredCardIndex, setHoveredCardIndex] = useState(null);
  // const [avg, totalAvg] = useState(null);
  const images = [img1, img2];
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(
          "https://68y0i55pli.execute-api.eu-west-2.amazonaws.com/PROD"
        );
        const responseData = await response.json();
        console.log(responseData);
        let avgTemp = 0;
        for (let i = 0; i < responseData.body.length; i++) {
          console.log(responseData.body[i][0]["DEPARTMENT"]);
          if (
            responseData.body[i][0]["DEPARTMENT"] === "Queries" ||
            responseData.body[i][0]["DEPARTMENT"] === "Reservation"
          ) {
            avgTemp = avgTemp + parseInt(responseData.body[i][0]["ANS_RATE"]);
            console.log(responseData.body[i][0]["ANS_RATE"]);
          }
        }
        console.log(avgTemp);
        props.setAvgValue(avgTemp / 2);
        setData(responseData.body.flat());
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };
    fetchData();
    const interval = setInterval(fetchData, 10000);
    return () => clearInterval(interval);
  }, []);
  const renderCard = (label, value, index) => (
    <div
      key={index}
      style={{
        ...cardStyle,
        ...(hoveredCardIndex === index ? hoveredCardStyle : null),
      }}
      onMouseEnter={() => handleMouseEnter(index)}
      onMouseLeave={() => handleMouseLeave()}
    >
      <span>{label}:</span>
      <span>{value}</span>
    </div>
  );
  const handleMouseEnter = (index) => {
    setHoveredCardIndex(index);
  };
  const handleMouseLeave = () => {
    setHoveredCardIndex(null);
  };
  const renderColumn = (items, department) => {
    return (
      <div style={columnStyle}>
        {items.map((item, index) => {
          const orderedItems = orderedKeys.map((key) => ({
            key: keyTranslations[key] || key,
            value: item[key],
          }));
          return orderedItems.map((orderedItem, subIndex) =>
            renderCard(
              orderedItem.key,
              orderedItem.value,
              `${department}-${index}-${subIndex}`
            )
          );
        })}
      </div>
    );
  };
  const queryItems = data.filter((item) => item.DEPARTMENT === "Queries");
  const reservationItems = data.filter(
    (item) => item.DEPARTMENT === "Reservation"
  );

  // console.log(avg, "avg");
  return (
    <div style={containerStyle}>
      {data.length > 0 && (
        <div style={dataContainerStyle}>
          {renderColumn(reservationItems, "Reservation")}
          {renderColumn(queryItems, "Query")}
        </div>
      )}
    </div>
  );
};
export default App;
