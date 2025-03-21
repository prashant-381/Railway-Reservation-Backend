require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");

const app = express();
app.use(express.json());
app.use(cors());

// Connect to MongoDB (Use MongoDB Atlas if you don’t have a local database)
mongoose.connect(process.env.MONGO_URI || "mongodb+srv://your_username:your_password@cluster.mongodb.net/railway", {
    useNewUrlParser: true,
    useUnifiedTopology: true,
}).then(() => console.log("MongoDB Connected")).catch(err => console.log(err));

// User Schema
const UserSchema = new mongoose.Schema({
    username: String,
    email: String,
    password: String,
});

const User = mongoose.model("User", UserSchema);

// Train Schema
const TrainSchema = new mongoose.Schema({
    number: String,
    name: String,
    from: String,
    to: String,
    time: String,
    price: Number,
});

const Train = mongoose.model("Train", TrainSchema);

// User Registration
app.post("/register", async (req, res) => {
    const { username, email, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({ username, email, password: hashedPassword });

    try {
        await user.save();
        res.status(201).json({ message: "User Registered" });
    } catch (error) {
        res.status(500).json({ error: "Error Registering User" });
    }
});

// User Login
app.post("/login", async (req, res) => {
    const { email, password } = req.body;
    const user = await User.findOne({ email });

    if (!user) return res.status(400).json({ error: "User Not Found" });

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) return res.status(400).json({ error: "Invalid Credentials" });

    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET, { expiresIn: "1h" });
    res.json({ token });
});

// Get Available Trains
app.get("/trains", async (req, res) => {
    const { from, to } = req.query;
    const trains = await Train.find({ from, to });

    if (trains.length === 0) return res.status(404).json({ error: "No Trains Found" });
    res.json(trains);
});

// Payment (Dummy API)
app.post("/payment", (req, res) => {
    const { amount, paymentMethod } = req.body;
    if (!amount || !paymentMethod) return res.status(400).json({ error: "Invalid Payment Details" });

    res.json({ message: "Payment Successful", transactionId: Math.random().toString(36).slice(2) });
});

// Start Server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
