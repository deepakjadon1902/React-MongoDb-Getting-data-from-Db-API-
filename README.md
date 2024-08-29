React:(Register.tsx):-
import { useState } from "react";
import './axios'
import api from "./axios";
const Register = (props) => {
  const [message, setMessage] = useState(false);
  const [data, setData] = useState({ 
    username: "", 
    email: "", 
    phonenumber: "", 
    password: "", 
    confirmPassword: "" 
  });
  const [error, setError] = useState(null);

  const updateData = (type, event) => {
    setData({ ...data, [type]: event.target.value });
  };
  const handleRegister = async (e) => {
    e.preventDefault();
    const response = await api.post("/register",data )
    console.log(data);
    const result = await response.data;
    console.log('result: ', result)
    setMessage("Registration successful!");
    console.log(result);
  };
  return (
    <div style={{ padding: "20px", textAlign: "center" }}>
      <h1>Register Form</h1>
      <div>
        <label>
          Username:
          <input
            type="text"
            value={data.username}
            onChange={(e) => updateData("username", e)}
          />
        </label>
      </div>
      <br />
      <div>
        <label>
          Email:
          <input
            type="email"
            value={data.email}
            onChange={(e) => updateData("email", e)}
          />
        </label>
      </div>
      <br />
      <div>
        <label>
          phonenumber:
          <input
            type="number"
            value={data.phonenumber}
            onChange={(e) => updateData("phonenumber", e)}
          />
        </label>
      </div>
      <br />
      <div>
        <label>
          Password:
          <input
            type="password"
            value={data.password}
            onChange={(e) => updateData("password", e)}
          />
        </label>
      </div>
      <br />
      {error && <p style={{ color: "red" }}>{error}</p>}
      {message && <p style={{ color: "green" }}>{message}</p>}
      <button onClick={handleRegister}>Register</button>
    </div>
  );
};
export default Register;



(table.tsx):-
import { useState } from "react";
import "./App.css"
import api from "./axios";
export const Table = (props) => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [sidebarOpen, setSidebarOpen] = useState(false); 
  const [selectedRow, setSelectedRow] = useState(null); 
  const getData = async () => {
    setLoading(true);
    const response = await api("/getdata");
    const responseValue = await response.data;
    setLoading(false);
    setData(responseValue);
  };

  const handleRemove = (row) => {
    setData(data.filter((r) => r.id !== row.id));
  };

  const handleEdit = (row) => {
    setSelectedRow(row);
    setSidebarOpen(true);
  };

  const handleCloseSidebar = () => {
    setSidebarOpen(false);
  };
  return (
    <>
      <button onClick={() => getData()}>Get Data</button>
      <button onClick={() => setData([])}>Reset</button>
      <table>
        <tr>
          <th>S.No</th>
          <th>User Name</th>
          <th>E-mail</th>
          <th>Phone</th>
          <th>password</th>
          <th>Action</th>
        </tr>
        {data.map((user, index) => (
          <tr key={index}>
            <td>{index + 1}</td>
            <td>{user.name}</td>
            <td>{user.email}</td>
            <td>{user.phonenumber}</td>
            <td>{user.password}</td>
            <td>
            <button onClick={() => handleEdit(user)}>Edit</button>
            {/* <button onClick={() => handleRemove(user)}>Remove</button> */}
            </td>
          </tr>
        ))}
      </table>
      {sidebarOpen && (
        <div style={{ position: "fixed", top: 0, right: 0, width: "300px", height: "100vh", backgroundColor: "#f0f0f0", padding: "20px", zIndex: 1000, color: "#000"}}>
          <h2>Row Details</h2>
          <p>Username: {selectedRow.name}</p>
          <p>Email: {selectedRow.email}</p>
          <p>Phonenumber: {selectedRow.phonenumber}</p>
          <button onClick={handleCloseSidebar}>Close</button>
        </div>
      )}
    </>
  );
};
export default Table;


MongoDb:(hello.py):-
from flask import Flask,request
from pymongo import MongoClient


from flask_cors import CORS   #pip install flask_cors

app = Flask(__name__)

cors=CORS(app,resources={r"/*":{"origins":"*"}})

client=MongoClient('mongodb://localhost:27017/')

mdb=client['pro456']

@app.route("/register", methods=['POST'])
def register():
    input_data = request.get_json()
    name = input_data.get('username')
    email = input_data.get('email')
    password = input_data.get('password')
    phonenumber = input_data.get('phonenumber')
    user = mdb.users.find_one({'email':email})
    print(user)
    if user and '_id' in user:
        return {
            'status': 0,
            'msg':"user exit",
            'class':"success"
        }
    count = mdb.users.count_documents({})
    users={
        "_id":count+1,
        "name":name,
        "email":email,
        "password":password,
        "Phonenumber" : phonenumber,
    }
    mdb.users.insert_one(users)
    return "inserted successfully"

@app.route("/getdata")
def getData():
    users = []
    for user in mdb.users.find():
        users.append(user)
    return users






