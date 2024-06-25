# Assignment
1. **Setup React Project**
   - Use Create React App to initialize a new project.

    ```bash
    npx create-react-app recipe-organizer
    cd recipe-organizer
    npm install axios react-router-dom
    ```

2. **Create Context for User Authentication**

    ```javascript
    // src/context/AuthContext.js
    import React, { createContext, useState, useEffect } from 'react';
    import axios from 'axios';

    const AuthContext = createContext();

    const AuthProvider = ({ children }) => {
      const [user, setUser] = useState(null);

      useEffect(() => {
        const token = localStorage.getItem('token');
        if (token) {
          axios.get('/me', { headers: { Authorization: token } })
            .then(response => setUser(response.data))
            .catch(() => localStorage.removeItem('token'));
        }
      }, []);

      const login = async (username, password) => {
        const response = await axios.post('/login', { username, password });
        localStorage.setItem('token', response.data.token);
        setUser(response.data.user);
      };

      const logout = () => {
        localStorage.removeItem('token');
        setUser(null);
      };

      return (
        <AuthContext.Provider value={{ user, login, logout }}>
          {children}
        </AuthContext.Provider>
      );
    };

    export { AuthProvider, AuthContext };
    ```

3. **React Components and Routes**

    ```javascript
    // src/App.js
    import React from 'react';
    import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
    import { AuthProvider } from './context/AuthContext';
    import Home from './components/Home';
    import Login from './components/Login';
    import Signup from './components/Signup';
    import AddRecipe from './components/AddRecipe';
    import PrivateRoute from './components/PrivateRoute';

    const App = () => {
      return (
        <AuthProvider>
          <Router>
            <Switch>
              <Route path="/login" component={Login} />
              <Route path="/signup" component={Signup} />
              <PrivateRoute path="/add-recipe" component={AddRecipe} />
              <PrivateRoute path="/" component={Home} />
            </Switch>
          </Router>
        </AuthProvider>
      );
    };

    export default App;
    ```

    ```javascript
    // src/components/PrivateRoute.js
    import React, { useContext } from 'react';
    import { Route, Redirect } from 'react-router-dom';
    import { AuthContext } from '../context/AuthContext';

    const PrivateRoute = ({ component: Component, ...rest }) => {
      const { user } = useContext(AuthContext);

      return (
        <Route
          {...rest}
          render={props =>
            user ? <Component {...props} /> : <Redirect to="/login" />
          }
        />
      );
    };

    export default PrivateRoute;
    ```

    ```javascript
    // src/components/Login.js
    import React, { useState, useContext } from 'react';
    import { AuthContext } from '../context/AuthContext';
    import { useHistory } from 'react-router-dom';

    const Login = () => {
      const [username, setUsername] = useState('');
      const [password, setPassword] = useState('');
      const { login } = useContext(AuthContext);
      const history = useHistory();

      const handleSubmit = async (e) => {
        e.preventDefault();
        await login(username, password);
        history.push('/');
      };

      return (
        <form onSubmit={handleSubmit}>
          <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Username" />
          <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" />
          <button type="submit">Login</button>
        </form>
      );
    };

    export default Login;
    ```

    ```javascript
    // src/components/Signup.js
    import React, { useState } from 'react';
    import axios from 'axios';
    import { useHistory } from 'react-router-dom';

    const Signup = () => {
      const [username, setUsername] = useState('');
      const [password, setPassword] = useState('');
      const history = useHistory();

      const handleSubmit = async (e) => {
        e.preventDefault();
        await axios.post('/signup', { username, password });
        history.push('/login');
      };

      return (
        <form onSubmit={handleSubmit}>
          <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Username" />
          <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" />
          <button type="submit">Signup</button>
        </form>
      );
    };

    export default Signup;
    ```

    ```javascript
    // src/components/AddRecipe.js
    import React, { useState, useContext } from 'react';
    import axios from 'axios';
    import { AuthContext } from '../context/AuthContext';

    const AddRecipe = () => {
      const [title, setTitle] = useState('');
      const [category, setCategory] = useState('');
      const [instructions, setInstructions] = useState('');
      const [image, setImage] = useState(null);
      const { user } = useContext(AuthContext);

      const handleSubmit = async (e) => {
        e.preventDefault();
        const formData = new FormData
