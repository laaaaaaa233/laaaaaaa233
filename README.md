const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();
const port = 3000;

app.use(bodyParser.json());
app.use(cors());

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/hotelReservation', { useNewUrlParser: true, useUnifiedTopology: true });

// Schema definitions
const guestSchema = new mongoose.Schema({
    name: String,
    contact_number: String
});

const roomSchema = new mongoose.Schema({
    room_number: String,
    type: String,
    price_per_night: Number,
    availability: Boolean
});

const serviceSchema = new mongoose.Schema({
    service_name: String,
    service_price: Number
});

const reservationSchema = new mongoose.Schema({
    guest_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Guest' },
    room_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Room' },
    checkin_date: Date,
    checkout_date: Date,
    total_amount: Number,
    services: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Service' }]
});

const Guest = mongoose.model('Guest', guestSchema);
const Room = mongoose.model('Room', roomSchema);
const Service = mongoose.model('Service', serviceSchema);
const Reservation = mongoose.model('Reservation', reservationSchema);

// API endpoints
app.post('/api/reservations', async (req, res) => {
    try {
        const guest = await Guest.create(req.body.guest);
        const reservation = new Reservation({
            guest_id: guest._id,
            room_id: req.body.room_id,
            checkin_date: req.body.checkin_date,
            checkout_date: req.body.checkout_date,
            total_amount: req.body.total_amount,
            services: req.body.services
        });

        await reservation.save();
        res.status(201).send(reservation);
    } catch (error) {
        res.status(500).send(error);
    }
});

app.get('/api/reservations', async (req, res) => {
    try {
        const reservations = await Reservation.find().populate('guest_id').populate('room_id').populate('services');
        res.status(200).send(reservations);
    } catch (error) {
        res.status(500).send(error);
    }
});

app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});
