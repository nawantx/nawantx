const express = require('express');
const app = express();
const path = require('path');
const fs = require('fs');

app.use(express.json());

const IPerson = {
  id: number,
  firstName: string,
  lastName: string,
  phoneNumber: string
};

const filePath = path.join(__dirname, 'people.json'); 
let contactList = [];

function loadContacts() {
  try {
    const rawData = fs.readFileSync(filePath, 'utf-8');
    contactList = JSON.parse(rawData);
  } catch (err) {
    console.error('Unable to read file: ' + filePath);
    contactList = [];
  }
}

loadContacts(); 

app.get('/contacts', (req, res) => {
  loadContacts(); 
  res.json(contactList);
});

app.get('/contacts/:id', (req, res) => {
  loadContacts(); // 
  const id = parseInt(req.params.id);
  const contact = contactList.find((c) => c.id === id);

  if (contact) {
    res.json(contact);
  } else {
    res.status(404).send('Contact not found');
  }
});

app.post('/contacts', (req, res) => {
  loadContacts(); // Reload contacts from the file
  const newContact = req.body;
  newContact.id = contactList.length + 1;
  contactList.push(newContact);

  fs.writeFileSync(filePath, JSON.stringify(contactList, null, 2));

  res.json(newContact);
});

app.put('/contacts/:id', (req, res) => {
  loadContacts(); // Reload contacts from the file
  const id = parseInt(req.params.id);
  const contactIndex = contactList.findIndex((c) => c.id === id);

  if (contactIndex === -1) {
    res.status(404).send('Contact not found');
  } else {
    const updatedContact = { ...contactList[contactIndex], ...req.body };
    contactList[contactIndex] = updatedContact;

    fs.writeFileSync(filePath, JSON.stringify(contactList, null, 2));

    res.json(updatedContact);
  }
});

app.delete('/contacts/:id', (req, res) => {
  loadContacts(); 
  const id = parseInt(req.params.id);
  const contactIndex = contactList.findIndex((c) => c.id === id);

  if (contactIndex === -1) {
    res.status(404).send('Contact not found');
  } else {
    contactList.splice(contactIndex, 1);

    fs.writeFileSync(filePath, JSON.stringify(contactList, null, 2));

    res.send('Contact deleted successfully');
  }
});

const port = process.env.PORT || 3000; 


app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});


