.card {
  display: flex;
  flex-wrap: wrap;
  border: 1px solid #ccc;
  padding: 12px;
  margin: 12px;
  max-width: 450px;
  max-height: 150px;
  border-radius: 10px;
  overflow: hidden;
}

.left-side {
  flex: 1;
  padding-right: 12px;
  text-align: center;
}

.profile-icon {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  font-size: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #ccc;
  color: white;
}

.name {
  font-size: 12px;
  margin-top: 4px;
}

.chat {
  color: #ccc;
  margin-top: 4px;
  font-size: 10px;
}

.time {
  margin-top: 4px;
  font-weight: bold;
  font-size: 14px;
}

.end-button {
  background-color: red;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 6px 12px;
  margin-top: 6px;
  cursor: pointer;
}

.right-side {
  flex: 2;
  padding-left: 12px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 6px;
}

.edit-icon,
.delete-icon {
  cursor: pointer;
  font-size: 14px;
}

.info {
  margin-top: 12px;
}

.info-line {
  flex: 1;
  display: flex;
  /* flex-direction: column; */
  margin-bottom: 6px;
  display: flex;
}

.info-label {
  font-weight: bold;
  font-size: 10px;
  margin-right: 4px;
}

.info-value {
  font-size: 10px;
  display: flex;
}

.grayed-out {
  color: grey;
}
