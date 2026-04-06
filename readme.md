// server.js
import express from "express";
import mongoose from "mongoose";
import jwt from "jsonwebtoken";
import cors from "cors";

const app = express();
app.use(express.json());
app.use(cors());

mongoose.connect("mongodb://127.0.0.1:27017/devtrack");

const UserSchema = new mongoose.Schema({
  email: String,
  password: String,
});

const SessionSchema = new mongoose.Schema({
  userId: String,
  hours: Number,
  date: { type: Date, default: Date.now },
});

const User = mongoose.model("User", UserSchema);
const Session = mongoose.model("Session", SessionSchema);

// Middleware Auth
const auth = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).send("No token");

  try {
    const decoded = jwt.verify(token, "secret");
    req.userId = decoded.id;
    next();
  } catch {
    res.status(401).send("Invalid token");
  }
};

// Registro
app.post("/register", async (req, res) => {
  const user = await User.create(req.body);
  res.json(user);
});

// Login
app.post("/login", async (req, res) => {
  const user = await User.findOne(req.body);
  if (!user) return res.status(404).send("User not found");

  const token = jwt.sign({ id: user._id }, "secret");
  res.json({ token });
});

// Criar sessão de estudo
app.post("/session", auth, async (req, res) => {
  const session = await Session.create({
    userId: req.userId,
    hours: req.body.hours,
  });
  res.json(session);
});

// Buscar sessões
app.get("/sessions", auth, async (req, res) => {
  const sessions = await Session.find({ userId: req.userId });
  res.json(sessions);
});

app.listen(3000, () => console.log("🚀 Server rodando"));
