# js
making node js
step 1 dependencies

mkdir secure-api
cd secure-api
npm init -y

step 2 database

npm install express mongoose bcrypt jsonwebtoken cors body-parser

step 3
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
const port = process.env.PORT || 3000;

// Connect to MongoDB (replace with your connection string)
mongoose.connect('mongodb://your-connection-string', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Create MongoDB models for User and Post (you should define your schemas)
const User = mongoose.model('User', { username: String, password: String });
const Post = mongoose.model('Post', { title: String, content: String });

app.use(cors());
app.use(bodyParser.json());

// User registration endpoint
app.post('/register', async (req, res) => {
  try {
    const { username, password } = req.body;

    // Hash the password
    const hashedPassword = await bcrypt.hash(password, 10);

    // Create a new user
    const user = new User({ username, password: hashedPassword });
    await user.save();

    res.status(201).json({ message: 'User registered successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
});

// User login endpoint
app.post('/login', async (req, res) => {
  try {
    const { username, password } = req.body;

    // Find the user by username
    const user = await User.findOne({ username });

    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Compare passwords
    const passwordMatch = await bcrypt.compare(password, user.password);

    if (!passwordMatch) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Generate a JWT token
    const token = jwt.sign({ userId: user._id }, 'your-secret-key', {
      expiresIn: '1h', // Token expiration time
    });

    res.json({ token });
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Post endpoints (requires authentication)
app.post('/posts', (req, res) => {
  // Add code to create a new post
});

app.get('/posts', (req, res) => {
  // Add code to retrieve posts
});

app.delete('/posts/:id', (req, res) => {
  // Add code to delete a post by ID
});

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});


step 4
node app.js


front end
step 1 registration
import React, { useState } from 'react';
import axios from 'axios';

const RegistrationForm = () => {
  const [formData, setFormData] = useState({
    username: '',
    password: '',
  });
  const [error, setError] = useState('');

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('/api/register', formData);
      // Handle successful registration (e.g., redirect to login page)
    } catch (err) {
      setError(err.response.data.error);
    }
  };

  return (
    <div>
      <h2>Registration</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          name="username"
          placeholder="Username"
          value={formData.username}
          onChange={handleChange}
          required
        />
        <input
          type="password"
          name="password"
          placeholder="Password"
          value={formData.password}
          onChange={handleChange}
          required
        />
        <button type="submit">Register</button>
      </form>
      {error && <div className="error">{error}</div>}
    </div>
  );
};

export default RegistrationForm;


strep 2  Displaying Posts Component (Posts.js):
import React, { useEffect, useState } from 'react';
import axios from 'axios';

const Posts = ({ authToken }) => {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    // Fetch posts from the backend API
    const fetchPosts = async () => {
      try {
        const response = await axios.get('/api/posts', {
          headers: {
            Authorization: `Bearer ${authToken}`,
          },
        });
        setPosts(response.data);
      } catch (error) {
        console.error('Error fetching posts:', error);
      }
    };

    fetchPosts();
  }, [authToken]);

  return (
    <div>
      <h2>Posts</h2>
      <ul>
        {posts.map((post) => (
          <li key={post._id}>
            <h3>{post.title}</h3>
            <p>{post.content}</p>
            <button onClick={() => deletePost(post._id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default Posts;

step 3 creating post
import React, { useState } from 'react';
import axios from 'axios';

const CreatePost = ({ authToken }) => {
  const [formData, setFormData] = useState({
    title: '',
    content: '',
  });

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await axios.post(
        '/api/posts',
        { ...formData },
        {
          headers: {
            Authorization: `Bearer ${authToken}`,
          },
        }
      );
      // Handle successful post creation (e.g., refresh the posts list)
    } catch (error) {
      console.error('Error creating post:', error);
    }
  };

  return (
    <div>
      <h2>Create Post</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          name="title"
          placeholder="Title"
          value={formData.title}
          onChange={handleChange}
          required
        />
        <textarea
          name="content"
          placeholder="Content"
          value={formData.content}
          onChange={handleChange}
          required
        />
        <button type="submit">Create Post</button>
      </form>
    </div>
  );
};

export default CreatePost;

step 4
import React, { useState } from 'react';
import axios from 'axios';

const LoginForm = ({ setAuthToken }) => {
  const [formData, setFormData] = useState({
    username: '',
    password: '',
  });
  const [error, setError] = useState('');

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('/api/login', formData);
      const { token } = response.data;
      setAuthToken(token);
      // Handle successful login (e.g., redirect to posts page)
    } catch (error) {
      setError(error.response.data.error);
    }
  };

  return (
    <div>
      <h2>Login</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          name="username"
          placeholder="Username"
          value={formData.username}
          onChange={handleChange}
          required
        />
        <input
          type="password"
          name="password"
          placeholder="Password"
          value={formData.password}
          onChange={handleChange}
          required
        />
        <button type="submit">Login</button>
      </form>
      {error && <div className="error">{error}</div>}
    </div>
  );
};

export default LoginForm;



