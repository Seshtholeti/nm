import React, { useEffect, useState } from "react";
import Header from "./Header";

const columnStyle = {
  flex: 1,
  display: "flex",
  flexDirection: "column",
  marginTop: "10px",
  marginLeft: "10px",
  gap: "12px",
  justifyContent: "center",
  alignItems: "center",
};

const containerStyle = {
  display: "flex",
  alignItems: "flex-start",
  color: "#333",
  padding: "20px",
  height: "80.9vh",
  boxSizing: "border-box",
  overflow: "hidden",
};
const imageContainerStyle = {
  width: "80%",
  height: "90%",
  display: "flex",
  justifyContent: "center",
  alignItems: "center",
  borderRadius: "5px",
  // backgroundColor: "red",
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
  height: "90%",
  // backgroundColor: "red",
  marginLeft: "10px",
  marginTop: "20px",
};

const cardStyle = {
  backgroundColor: "#00008B",
  padding: "8px",
  borderRadius: "5px",
  display: "flex",
  justifyContent: "space-between",
  alignItems: "center",
  height: "30px",
  color: "#fff",
  fontSize: "16px",
  width: "200px",
  transition: "background-color 0.3s ease",
};
const hoveredCardStyle = {
  backgroundColor: "red",
  color: "white",
  cursor: "pointer",
};
const App = () => {
  const [data, setData] = useState([]);
  const [imageUrl, setImageUrl] = useState("");
  const [hoveredCardIndex, setHoveredCardIndex] = useState(null); // Track hovered state for each card
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(
          "https://0ns1q7m4zj.execute-api.eu-west-2.amazonaws.com/de"
        );
        const data = await response.json();
        console.log(data);
        setData(data.body);
        setImageUrl(
          "https://wb-quicksight-html.s3.eu-west-2.amazonaws.com/Whitbread-image.jpg"
        ); // Replace with the actual URL
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };
    // Fetch data initially
    fetchData();
    // Fetch data every 10 seconds (adjust interval as needed)
    const interval = setInterval(fetchData, 10000);
    // Cleanup interval on component unmount
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
  const renderColumn = (items, department) => (
    <div style={columnStyle}>
      {items.map((item, index) =>
        Object.keys(item.Item).map((key, subIndex) =>
          renderCard(key, item.Item[key], `${department}-${index}-${subIndex}`)
        )
      )}
    </div>
  );
  const queryItems = data.filter((item) => item.Item.DEPARTMENT === "Query");
  const reservationItems = data.filter(
    (item) => item.Item.DEPARTMENT === "Reservation"
  );
  const groupItems = data.filter((item) => item.Item.DEPARTMENT === "Group");
  return (
    <div style={containerStyle}>
      <div style={imageContainerStyle}>
        {imageUrl ? (
          <img src={imageUrl} alt="Uploaded" style={imageStyle} />
        ) : (
          <span>Upload Image</span>
        )}
      </div>
      {data.length > 0 && (
        <div style={dataContainerStyle}>
          {renderColumn(queryItems, "Query")}
          {renderColumn(reservationItems, "Reservation")}
          {renderColumn(groupItems, "Group")}
        </div>
      )}
    </div>
  );
};
export default App;




this is the api

https://9lczoy9kqi.execute-api.eu-west-2.amazonaws.com/uk

{"statusCode":200,"body":[[{"ANS_RATE":"0","OFFERED":0,"ANS":0,"RDY":0,"ONLINE":0,"NOT_RDY":0,"TALK":0,"LWT":0,"CIQ":0,"DEPARTMENT":"Reservation center"}],[{"ANS_RATE":"0","OFFERED":0,"ANS":0,"RDY":0,"ONLINE":0,"NOT_RDY":0,"TALK":0,"LWT":0,"CIQ":0,"DEPARTMENT":"Guest Relations"}],[{"ANS_RATE":"0","OFFERED":0,"ANS":0,"RDY":0,"ONLINE":0,"NOT_RDY":0,"TALK":0,"LWT":0,"CIQ":0,"DEPARTMENT":"Restaurant"}]]}


