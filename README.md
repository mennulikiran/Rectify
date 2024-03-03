# Rectify
"React and Node.js app with PostgreSQL database. Displays 50 customer records with search, pagination, and sorting functionalities. Simplify data management and analysis."
**SQL CODE to setup the PostgreSQL database:**

CREATE TABLE customers (
  sno SERIAL PRIMARY KEY,
  customer_name VARCHAR(255) NOT NULL,
  age INT,
  phone VARCHAR(20),
  location VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Generate dummy data (replace with your preferred method)
INSERT INTO customers (customer_name, age, phone, location)
VALUES ('John Doe', 30, '123-456-7890', 'New York'),
       ('Jane Smith', 25, '987-654-3210', 'Los Angeles'),
       ... (add more records as needed);
**Set up the React frontend**
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [customers, setCustomers] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState('');
  const [page, setPage] = useState(1);

  useEffect(() => {
    fetchData();
  }, [searchTerm, sortBy, page]);

  const fetchData = async () => {
    try {
      const response = await axios.get(`/api/customers?page=${page}&search=${searchTerm}&sortBy=${sortBy}`);
      setCustomers(response.data);
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  };

  const handleSearchChange = (event) => {
    setSearchTerm(event.target.value);
  };

  const handleSortChange = (event) => {
    setSortBy(event.target.value);
  };

  const handlePageChange = (newPage) => {
    setPage(newPage);
  };

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={handleSearchChange}
        placeholder="Search by name or location"
      />
      <select value={sortBy} onChange={handleSortChange}>
        <option value="">Sort By</option>
        <option value="date">Date</option>
        <option value="time">Time</option>
      </select>
      <table>
        <thead>
          <tr>
            <th>S.No</th>
            <th>Customer Name</th>
            <th>Age</th>
            <th>Phone</th>
            <th>Location</th>
            <th>Date</th>
            <th>Time</th>
          </tr>
        </thead>
        <tbody>
          {customers.map((customer) => (
            <tr key={customer.sno}>
              <td>{customer.sno}</td>
              <td>{customer.customer_name}</td>
              <td>{customer.age}</td>
              <td>{customer.phone}</td>
              <td>{customer.location}</td>
              <td>{new Date(customer.created_at).toLocaleDateString()}</td>
              <td>{new Date(customer.created_at).toLocaleTimeString()}</td>
            </tr>
          ))}
        </tbody>
      </table>
      <button onClick={() => handlePageChange(page - 1)} disabled={page === 1}>Previous</button>
      <button onClick={() => handlePageChange(page + 1)}>Next</button>
    </div>
  );
}

export default App;

**Set up the Node.js backend:**
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const db = require('./db'); // Assuming a separate `db.js` module for database connection

const app = express();
const PORT = process.env.PORT || 5000;

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(cors());

// Fetch 20 records per page
const pageSize = 20;

// API endpoint to get paginated, sorted, and filtered customer data
app.get('/api/customers', async (req, res) => {
  const { page = 1, search, sortBy } = req.query;
  const offset = (page - 1) * pageSize;

  let query = `SELECT * FROM customers`;

  if (search) {
    query += ` WHERE LOWER(customer_name) LIKE LOWER('%<span class="math-inline">\{search\}%'\) OR LOWER\(location\) LIKE LOWER\('%</span>{search}%')`;
  }

  if (sortBy === 'date') {
    query += ` ORDER BY DATE(created_at)`;
  } else if (sortBy === 'time') {
    query += ` ORDER BY TIME(created_at)`;
  }

  query += ` LIMIT ${pageSize} OFFSET ${offset}`;

  try {
    const customers = await db.query(query);
    res.json(customers.rows); // Assuming `db.query` returns an object with rows property
  } catch (error) {
    console.error('Error fetching customers:', error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});
