## Hi there 👋

<!--
**NaruMichy/NaruMichy** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
Projeto Sugar Book — Completo (React + Supabase + Pandinhas fofos)

1. package.json



{
"name": "sugar-book",
"version": "1.0.0",
"private": true,
"dependencies": {
"@supabase/supabase-js": "^2.0.0",
"react": "^18.0.0",
"react-dom": "^18.0.0",
"react-router-dom": "^6.3.0"
},
"scripts": {
"start": "react-scripts start",
"build": "react-scripts build"
}
}


---

2. src/index.js



import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);


---

3. src/App.js



import React from "react";
import { BrowserRouter as Router, Routes, Route, Navigate } from "react-router-dom";
import { AuthProvider, useAuth } from "./lib/AuthContext";
import Home from "./pages/Home";
import Login from "./pages/Login";
import Signup from "./pages/Signup";
import Dashboard from "./pages/Dashboard";
import Wishlist from "./pages/Wishlist";
import PixPage from "./pages/PixPage";

function PrivateRoute({ children }) {
const { user } = useAuth();
return user ? children : <Navigate to="/login" />;
}

export default function App() {
return (
<AuthProvider>
<Router>
<Routes>
<Route path="/" element={<Home />} />
<Route path="/login" element={<Login />} />
<Route path="/signup" element={<Signup />} />
<Route
path="/dashboard"
element={
<PrivateRoute>
<Dashboard />
</PrivateRoute>
}
/>
<Route
path="/wishlist"
element={
<PrivateRoute>
<Wishlist />
</PrivateRoute>
}
/>
<Route
path="/patrocinar"
element={
<PrivateRoute>
<PixPage />
</PrivateRoute>
}
/>
</Routes>
</Router>
</AuthProvider>
);
}


---

4. src/lib/AuthContext.js



import React, { createContext, useContext, useState, useEffect } from "react";
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = "https://seu-projeto.supabase.co";
const supabaseKey = "public-anon-key";
const supabase = createClient(supabaseUrl, supabaseKey);

const AuthContext = createContext();

export function AuthProvider({ children }) {
const [user, setUser] = useState(null);

useEffect(() => {
const session = supabase.auth.getSession().then(({ data: { session } }) => {
setUser(session?.user ?? null);
});

const { data: listener } = supabase.auth.onAuthStateChange((_event, session) => {  
  setUser(session?.user ?? null);  
});  

return () => {  
  listener.unsubscribe();  
};

}, []);

const login = async (email, password) => {
const { error, data } = await supabase.auth.signInWithPassword({ email, password });
if (error) throw error;
setUser(data.user);
};

const signup = async (email, password) => {
const { error, data } = await supabase.auth.signUp({ email, password });
if (error) throw error;
setUser(data.user);
};

const logout = async () => {
await supabase.auth.signOut();
setUser(null);
};

return (
<AuthContext.Provider value={{ user, login, signup, logout }}>
{children}
</AuthContext.Provider>
);
}

export function useAuth() {
return useContext(AuthContext);
}


---

5. src/pages/Home.js



import React from "react";
import { Link } from "react-router-dom";
import panda from "../assets/panda.png";

export default function Home() {
return (
<div style={{ textAlign: "center", padding: "2rem" }}>
<img src={panda} alt="Panda fofinho" width={200} />
<h1>Sugar Book</h1>
<p>Porque todo leitor merece seu sugar.</p>
<Link to="/login">Login</Link> | <Link to="/signup">Cadastro</Link>
<p>Prévia para não logados: veja só como a página fica fofa com pandinhas e livros.</p>
</div>
);
}


---

6. src/pages/Login.js



import React, { useState } from "react";
import { useNavigate, Link } from "react-router-dom";
import { useAuth } from "../lib/AuthContext";

export default function Login() {
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");
const [error, setError] = useState(null);
const { login } = useAuth();
const navigate = useNavigate();

async function handleSubmit(e) {
e.preventDefault();
try {
await login(email, password);
navigate("/dashboard");
} catch (err) {
setError(err.message);
}
}

return (
<form onSubmit={handleSubmit} style={{ maxWidth: "400px", margin: "2rem auto" }}>
<h2>Login</h2>
<input type="email" placeholder="Email" value={email} onChange={e => setEmail(e.target.value)} required />
<input
type="password"
placeholder="Senha"
value={password}
onChange={e => setPassword(e.target.value)}
required
/>
{error && <p style={{ color: "red" }}>{error}</p>}
<button type="submit">Entrar</button>
<p>
Não tem conta? <Link to="/signup">Cadastre-se</Link>
</p>
</form>
);
}


---

7. src/pages/Signup.js



import React, { useState } from "react";
import { useNavigate, Link } from "react-router-dom";
import { useAuth } from "../lib/AuthContext";

export default function Signup() {
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");
const [error, setError] = useState(null);
const { signup } = useAuth();
const navigate = useNavigate();

async function handleSubmit(e) {
e.preventDefault();
try {
await signup(email, password);
navigate("/dashboard");
} catch (err) {
setError(err.message);
}
}

return (
<form onSubmit={handleSubmit} style={{ maxWidth: "400px", margin: "2rem auto" }}>
<h2>Cadastro</h2>
<input type="email" placeholder="Email" value={email} onChange={e => setEmail(e.target.value)} required />
<input
type="password"
placeholder="Senha"
value={password}
onChange={e => setPassword(e.target.value)}
required
/>
{error && <p style={{ color: "red" }}>{error}</p>}
<button type="submit">Cadastrar</button>
<p>
Já tem conta? <Link to="/login">Faça login</Link>
</p>
</form>
);
}


---

8. src/pages/Dashboard.js



import React from "react";
import { useAuth } from "../lib/AuthContext";
import { Link, useNavigate } from "react-router-dom";

export default function Dashboard() {
const { user, logout } = useAuth();
const navigate = useNavigate();

async function handleLogout() {
await logout();
navigate("/");
}

return (
<div style={{ padding: "2rem", textAlign: "center" }}>
<h2>Bem-vindo(a), {user.email}</h2>
<p>
<Link to="/wishlist">Sua Wishlist de Livros</Link>
</p>
<p>
<Link to="/patrocinar">Patrocinar com Pix</Link>
</p>
<button onClick={handleLogout}>Sair</button>
</div>
);
}


---

9. src/pages/Wishlist.js



import React, { useState, useEffect } from "react";
import { useAuth } from "../lib/AuthContext";

export default function Wishlist() {
const { user } = useAuth();
const [books, setBooks] = useState([]);
const [newBook, setNewBook] = useState("");

// Aqui você conecta com Supabase para buscar e salvar livros da wishlist
// Pra simplificar, tá só local por enquanto

useEffect(() => {
const stored = JSON.parse(localStorage.getItem("wishlist-" + user.email)) || [];
setBooks(stored);
}, [user]);

useEffect(() => {
localStorage.setItem("wishlist-" + user.email, JSON.stringify(books));
}, [books, user]);

function


---

💻 Componente completo com mensagem + pandinhas fofos

import React, { useState } from "react";
import QRCode from "qrcode.react";

export default function ApoioLeitorFofo() {
  const [valor, setValor] = useState("25.00");
  const [mensagem, setMensagem] = useState("");
  const [mostrarPix, setMostrarPix] = useState(false);
  const [erro, setErro] = useState("");

  const chavePix = "pixleitor@email.com";
  const nome = "Leitor Panda";
  const cidade = "Curitiba";

  const gerarPayloadPix = (chave, nome, cidade, valor) => {
    const val = parseFloat(valor).toFixed(2);
    return `00020126360014BR.GOV.BCB.PIX0114${chave}5204000053039865405${val.replace(".", "")}5802BR5911${nome}6012${cidade}62100506SB0016304`;
  };

  const handleGerarPix = () => {
    const v = parseFloat(valor);
    if (isNaN(v) || v < 25) {
      setErro("Valor mínimo: R$25,00");
      setMostrarPix(false);
      return;
    }
    setErro("");
    setMostrarPix(true);
  };

  const payload = gerarPayloadPix(chavePix, nome, cidade, valor);

  return (
    <div className="max-w-md mx-auto p-6 bg-white shadow-lg rounded-xl relative">
      {/* Pandinhas fofos pendurados */}
      <img
        src="https://i.imgur.com/IJOgqkT.png"
        alt="Panda"
        className="absolute -top-6 left-2 w-12 rotate-[10deg]"
      />
      <img
        src="https://i.imgur.com/IJOgqkT.png"
        alt="Panda"
        className="absolute -top-6 right-2 w-12 -rotate-[10deg]"
      />
      <h2 className="text-2xl font-bold text-center mb-4">Patrocine esse leitor 🐼</h2>
      <p className="text-gray-600 text-sm text-center mb-6">
        Leitor fofo merece carinho e incentivo! Deixe um recadinho e mande um Pix 💖
      </p>

      <div className="mb-4">
        <label className="text-sm font-medium">Valor (mínimo R$25):</label>
        <input
          type="number"
          className="w-full mt-1 border rounded px-3 py-2"
          value={valor}
          onChange={(e) => setValor(e.target.value)}
        />
      </div>

      <div className="relative mb-6">
        <label className="text-sm font-medium">Mensagem para o leitor:</label>
        <textarea
          rows="3"
          className="w-full mt-1 border rounded px-3 py-2 resize-none"
          placeholder="Sua leitura salvou minha semana... 🥺💕"
          value={mensagem}
          onChange={(e) => setMensagem(e.target.value)}
        />
        <img
          src="https://i.imgur.com/4P9fWDf.png"
          alt="Panda fofo"
          className="absolute -bottom-4 right-2 w-10"
        />
      </div>

      <button
        onClick={handleGerarPix}
        className="bg-indigo-600 text-white w-full py-2 rounded hover:bg-indigo-700"
      >
        Gerar QR Code do Pix
      </button>

      {erro && <p className="text-red-500 mt-3">{erro}</p>}

      {mostrarPix && (
        <div className="mt-6 text-center">
          <p className="text-sm mb-2">Escaneie o QR Code ou copie o código Pix:</p>
          <QRCode value={payload} size={180} className="mx-auto mb-3" />
          <div className="bg-gray-100 text-sm p-2 rounded break-all">{payload}</div>

          {mensagem && (
            <div className="mt-4 bg-yellow-50 border border-yellow-200 p-3 rounded shadow-inner">
              <p className="text-sm italic">“{mensagem}”</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

