# setupinstructions
git clone https://github.com/your-username/contact-management-crm.git
cd contact-management-crm
## Setup Instructions

### 1. Clone the repo:
```bash
git clone https://github.com/your-username/contact-management-crm.git
cd contact-management-crm

# For frontend
cd frontend
npm init -y
npm install react react-dom @mui/material axios
import React, { useState } from 'react';
import { TextField, Button, Container, Grid } from '@mui/material';
import axios from 'axios';

const ContactForm = () => {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    phone: '',
    company: '',
    jobTitle: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({ ...formData, [name]: value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await axios.post('http://localhost:5000/contacts', formData);
      alert('Contact added successfully!');
    } catch (err) {
      console.error(err);
      alert('Error adding contact');
    }
  };

  return (
    <Container>
      <h2>Add New Contact</h2>
      <form onSubmit={handleSubmit}>
        <Grid container spacing={2}>
          <Grid item xs={6}>
            <TextField
              fullWidth
              label="First Name"
              name="firstName"
              value={formData.firstName}
              onChange={handleChange}
              required
            />
          </Grid>
          <Grid item xs={6}>
            <TextField
              fullWidth
              label="Last Name"
              name="lastName"
              value={formData.lastName}
              onChange={handleChange}
              required
            />
          </Grid>
          <Grid item xs={12}>
            <TextField
              fullWidth
              label="Email"
              name="email"
              value={formData.email}
              onChange={handleChange}
              required
            />
          </Grid>
          <Grid item xs={12}>
            <TextField
              fullWidth
              label="Phone"
              name="phone"
              value={formData.phone}
              onChange={handleChange}
              required
            />
          </Grid>
          <Grid item xs={12}>
            <TextField
              fullWidth
              label="Company"
              name="company"
              value={formData.company}
              onChange={handleChange}
            />
          </Grid>
          <Grid item xs={12}>
            <TextField
              fullWidth
              label="Job Title"
              name="jobTitle"
              value={formData.jobTitle}
              onChange={handleChange}
            />
          </Grid>
          <Grid item xs={12}>
            <Button type="submit" variant="contained">Add Contact</Button>
          </Grid>
        </Grid>
      </form>
    </Container>
  );
};

export default ContactForm;
import React, { useState, useEffect } from 'react';
import { Table, TableBody, TableCell, TableContainer, TableHead, TableRow, Button, TablePagination } from '@mui/material';
import axios from 'axios';

const ContactsTable = () => {
  const [contacts, setContacts] = useState([]);
  const [page, setPage] = useState(0);
  const [rowsPerPage, setRowsPerPage] = useState(5);

  useEffect(() => {
    const fetchContacts = async () => {
      const res = await axios.get('http://localhost:5000/contacts');
      setContacts(res.data);
    };
    fetchContacts();
  }, []);

  const handleDelete = async (id) => {
    await axios.delete(`http://localhost:5000/contacts/${id}`);
    setContacts(contacts.filter(contact => contact._id !== id));
  };

  const handleChangePage = (event, newPage) => {
    setPage(newPage);
  };

  const handleChangeRowsPerPage = (event) => {
    setRowsPerPage(parseInt(event.target.value, 10));
    setPage(0);
  };

  return (
    <TableContainer>
      <Table>
        <TableHead>
          <TableRow>
            <TableCell>First Name</TableCell>
            <TableCell>Last Name</TableCell>
            <TableCell>Email</TableCell>
            <TableCell>Phone</TableCell>
            <TableCell>Company</TableCell>
            <TableCell>Job Title</TableCell>
            <TableCell>Actions</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {contacts.slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage).map(contact => (
            <TableRow key={contact._id}>
              <TableCell>{contact.firstName}</TableCell>
              <TableCell>{contact.lastName}</TableCell>
              <TableCell>{contact.email}</TableCell>
              <TableCell>{contact.phone}</TableCell>
              <TableCell>{contact.company}</TableCell>
              <TableCell>{contact.jobTitle}</TableCell>
              <TableCell>
                <Button onClick={() => handleDelete(contact._id)} color="error">Delete</Button>
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
      <TablePagination
        rowsPerPageOptions={[5, 10, 25]}
        component="div"
        count={contacts.length}
        rowsPerPage={rowsPerPage}
        page={page}
        onPageChange={handleChangePage}
        onRowsPerPageChange={handleChangeRowsPerPage}
      />
    </TableContainer>
  );
};

export default ContactsTable;
npm install
npm start



# For backend
cd ../backend
npm init -y
npm install express mongoose body-parser cors
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');
const Contact = require('./models/contact'); // Define this model later

const app = express();

// Middleware
app.use(cors());
app.use(bodyParser.json());

// MongoDB Connection (Example for MongoDB)
mongoose.connect('mongodb://localhost:27017/contacts', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => {
  console.log('MongoDB connected');
}).catch(err => console.log(err));

// Define Routes (API Endpoints)
app.post('/contacts', async (req, res) => {
  try {
    const { firstName, lastName, email, phone, company, jobTitle } = req.body;
    if (!firstName || !lastName || !email || !phone) {
      return res.status(400).send('All fields are required');
    }
    const newContact = new Contact(req.body);
    await newContact.save();
    res.status(201).send(newContact);
  } catch (err) {
    res.status(500).send('Server error');
  }
});

app.get('/contacts', async (req, res) => {
  try {
    const contacts = await Contact.find();
    res.json(contacts);
  } catch (err) {
    res.status(500).send('Server error');
  }
});

app.put('/contacts/:id', async (req, res) => {
  try {
    const updatedContact = await Contact.findByIdAndUpdate(req.params.id, req.body, { new: true });
    res.json(updatedContact);
  } catch (err) {
    res.status(500).send('Server error');
  }
});

app.delete('/contacts/:id', async (req, res) => {
  try {
    await Contact.findByIdAndDelete(req.params.id);
    res.status(204).send();
  } catch (err) {
    res.status(500).send('Server error');
  }
});

// Start Server
const PORT = 5000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
const mongoose = require('mongoose');

const contactSchema = new mongoose.Schema({
  firstName: { type: String, required: true },
  lastName: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  phone: { type: String, required: true },
  company: String,
  jobTitle: String,
});

module.exports = mongoose.model('Contact', contactSchema);
cd backend
node server.js
npm install
node server.js

