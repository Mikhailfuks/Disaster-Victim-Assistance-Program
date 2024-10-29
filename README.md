// models/Victim.js
const mongoose = require('mongoose');

const victimSchema = new mongoose.Schema({
    name: { type: String, required: true },
    age: { type: Number, required: true },
    location: { type: String, required: true },
    needs: { type: String, required: true },
}, { timestamps: true });

module.exports = mongoose.model('Victim', victimSchema);


// index.js
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');
require('dotenv').config();
const Victim = require('./models/Victim');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(bodyParser.json());
app.use(express.static('public'));

// Подключение к MongoDB
mongoose.connect(process.env.MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected successfully!'))
.catch(err => console.error('MongoDB connection error:', err));

// Маршрут для добавления жертвы
app.post('/add-victim', async (req, res) => {
    const { name, age, location, needs } = req.body;

    try {
        const victim = new Victim({ name, age, location, needs });
        await victim.save();
        res.status(201).json({ message: 'Victim added successfully!' });
    } catch (error) {
        res.status(500).json({ message: 'Error adding victim', error });
    }
});

// Маршрут для получения информации о жертвах
app.get('/victims', async (req, res) => {
    try {
        const victims = await Victim.find();
        res.status(200).json(victims);
    } catch (error) {
        res.status(500).json({ message: 'Error fetching victims', error });
    }
});

// Запускаем сервер
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Disaster Relief</title>
    <link rel="stylesheet" href="styles.css">
    <script defer src="app.js"></script>
</head>
<body>
    <div class="container">
        <h1>Disaster Relief Support</h1>
        <form id="victimForm">


            <input type="text" id="name" placeholder="Name" required>
            <input type="number" id="age" placeholder="Age" required>
            <input type="text" id="location" placeholder="Location" required>
            <textarea id="needs" placeholder="Needs" required></textarea>
            <button type="submit">Add Victim</button>
        </form>
        <h2>Victims List</h2>
        <ul id="victimsList"></ul>
        <p id="message"></p>
    </div>
</body>
</html>


/* public/styles.css */
body {
    font-family: Arial, sans-serif;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    height: 100vh;
    margin: 0;
    background-color: #f0f0f0;
}

.container {
    max-width: 600px;
    padding: 20px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

input, textarea {
    width: 100%;
    padding: 10px;
    margin: 5px 0;
    border: 1px solid #ccc;
    border-radius: 4px;
}

button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 10px;
    cursor: pointer;
    border-radius: 4px;
    transition: background 0.3s;
}

button:hover {
    background-color: #0056b3;
}

ul {
    list-style-type: none;
    padding: 0;
}

#message {
    color: green;
}



// public/app.js
document.getElementById('victimForm').addEventListener('submit', async function(event) {
    event.preventDefault();

    const name = document.getElementById('name').value;
    const age = document.getElementById('age').value;
    const location = document.getElementById('location').value;
    const needs = document.getElementById('needs').value;
    const messageElement = document.getElementById('message');

    try {
        const response = await fetch('/add-victim', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, age, location, needs })
        });

        if (response.ok) {
            messageElement.textContent = 'Victim added successfully!';
            messageElement.style.color = 'green';
            fetchVictims(); // Обновляем список жертв
            document.getElementById('victimForm').reset(); // Сброс форм
        } else {
            const errorData = await response.json();
            messageElement.textContent = errorData.message;
            messageElement.style.color = 'red';
        }
    } catch (error) {
        messageElement.textContent = 'An error occurred. Please try again.';
        messageElement.style.color = 'red';
    }
});

// Функция для получения списка жертв
async function fetchVictims() {
    const response = await fetch('/victims');
    const victims = await response.json();

    const victimsList = document.getElementById('victimsList');
    victimsList.innerHTML = ''; // Очищаем предыдущий список

    victims.forEach(victim => {
        const li = document.createElement('li');
        li.textContent = `${victim.name}, Age: ${victim.age}, Location: ${victim.location}, Needs: ${victim.needs}`;
        victimsList.appendChild(li);
    });
}

// Получаем список жертв при загрузке страницы
window.onload = fetchVictims;
