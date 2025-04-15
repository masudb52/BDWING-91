const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const authRoutes = require('./routes/auth');

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost:27017/okwin_clone', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

app.use('/api/auth', authRoutes);

app.listen(5000, () => {
  console.log('Server running on port 5000');
}); 
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: String,
  email: String,
  password: String,
  walletBalance: { type: Number, default: 100 },
});

module.exports = mongoose.model('User', userSchema);
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const User = require('../models/User');
const router = express.Router();

const SECRET = "OKWIN_SECRET_KEY";

router.post('/register', async (req, res) => {
  const { username, email, password } = req.body;
  const hash = await bcrypt.hash(password, 10);
  const user = new User({ username, email, password: hash });
  await user.save();
  res.send({ message: 'User registered' });
});

router.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user) return res.status(404).send({ error: 'User not found' });

  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(401).send({ error: 'Invalid credentials' });

  const token = jwt.sign({ id: user._id }, SECRET, { expiresIn: '1d' });
  res.send({ token, user: { username: user.username, wallet: user.walletBalance } });
});

module.exports = router;
# BDWING-91
