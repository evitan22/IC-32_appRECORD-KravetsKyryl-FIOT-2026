## Тема, Мета, Місце розташування

**Тема:** Laba3

**Мета:** 
- Вивчення принципів побудови REST API;
- Набуття практичних навичок розробки серверного застосунку з використанням платформи Node.js і фреймворку Express;
- Реалізувати механізми реєстрації та авторизації користувачів;
- Забезпечити валідацію вхідних даних;
- Забезпечити обробку помилок;
- Організувати захищений доступ до ресурсів із використанням JWT-токенів і системи ролей користувачів.

**Місце розташування:**
- [GitHub](https://github.com/evitan22/IC-32_appRECORD-KravetsKyryl-FIOT-2026)
- [Live demo](https://evitan22.github.io/IC-32_appRECORD-KravetsKyryl-FIOT-2026/)

---

## Хід виконання

### Пункт 2 - Реєстрація та авторизація користувача

#### Registration - Front code

```
const registrationForm = document.querySelector(".registrationForm");
registrationForm.addEventListener('submit', event => {
    event.preventDefault();

    const formValue = validateForm();

    sendRegistrationRequest(JSON.stringify(formValue));
});

function sendRegistrationRequest(data) {
    fetch("http://localhost:3000/register", {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: data,
    })
    .then(response => {
        if (!response.ok) {
            return response.text().then(text => { throw new Error(text) });
        }
        return response.json();
    })
    .then(response => {
        window.alert("You have been registrated. Now you should log in.");
        window.location.assign("index.html");
    })
    .catch(er => console.log(er));
}
```

#### Registration - Back code

```
app.post("/register", async (req, res) => {
  try {
    const { email, password, name } = req.body;

    // Валідація
    if (!email || !password) {
      return res.status(400).json({ message: "Всі поля обов'язкові" });
    }
    if (password.length < 6) {
      return res.status(400).json({ message: "Пароль занадто короткий" });
    }

    // Перевірка існування користувача
    const userExists = await User.findOne({ where: { email } });
    if (userExists) return res.status(400).json({ message: "Користувач вже існує" });

    // Хешування пароля
    const hashedPassword = await bcrypt.hash(password, 10);
    await User.create({ email, password: hashedPassword, name });

    res.status(201).json({ message: "Користувача створено" });
} catch (error) {
    console.error(error); 
    res.status(500).json({ message: "Помилка сервера" });
}
});
```

#### Authorization - Front code

```
const loginForm = document.querySelector(".loginForm");
console.log(loginForm);
loginForm.addEventListener('submit', event => {
    event.preventDefault();

    const formValue = validateForm();

    sendLoginRequest(JSON.stringify(formValue));
});

function sendLoginRequest(data) {
    fetch("http://localhost:3000/login", {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: data,
    })
    .then(response => {
        if (!response.ok) {
            return response.text().then(text => { throw new Error(text) });
        }
        return response.json();
    })
    .then(response => {
        console.log(response, "response");
        const jwtToken = response;
        localStorage.setItem("token", jwtToken.token);
        localStorage.setItem("refreshToken", jwtToken.refreshToken);
        window.location.assign("index.html");
    })
    .catch(er => console.log(er));
}
```

#### Authorization - Back code

```
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, 
    max: 2, 
    handler: (req, res, next, options) => {
    res.status(429).json({ 
        message: "Забагато невдалих спроб. Спробуйте через 15 хвилин." 
    });
    },
    standardHeaders: true, 
    legacyHeaders: false,
});

// Авторизація
app.post('/login', loginLimiter, async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ where: { email } });

    if (!user) {
      return res.status(400).json({ message: "Користувача не знайдено" });
    }

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      return res.status(400).json({ message: "Невірний пароль" });
    }

    const token = jwt.sign({ email, name: user.name, id: user.id }, SECRET_KEY, { expiresIn: "1m" });
    const refreshToken = jwt.sign({ email, name: user.name, id: user.id }, SECRET_KEY, { expiresIn: "1h" });
    res.json({ token, refreshToken });
  } catch (error) {
    res.status(500).json({ message: "Помилка сервера" });
  }
});
```

### Пункт 3 - Валідація та обробка помилок

```
if (!email || !password) {
    return res.status(400).json({ message: "Всі поля обов'язкові" });
}
if (password.length < 6) {
    return res.status(400).json({ message: "Пароль занадто короткий" });
}

if (!user) {
    return res.status(400).json({ message: "Користувача не знайдено" });
}

const isMatch = await bcrypt.compare(password, user.password);
if (!isMatch) {
    return res.status(400).json({ message: "Невірний пароль" });
}
```

### Пункт 4 - Реалізувати захищений маршрут

```
const authHeader = req.headers["authorization"];
```

### Пункт 5 - Протестувати API через Postman або curl

```
app.post("/register", async (req, res) => {...}
app.post('/login', loginLimiter, async (req, res) => {...}
```

### Пункт 7 - Додати підтвердження пароля при реєстрації

```
if (inputPassword2) {
    if (passwordValue1 == passwordValue2) {
        errorPassword2.style.display = "none";
    } else {
        errorPassword2.style.display = "block";
        return;
    }
}
```

### Пункт 9 - реалізувати log out

```
function displayRegistrationButtons() {
    loginBtn.style.display = "block";
    registrationBtn.style.display = "block";
    profileBtn.style.display = "none";
    logOut.style.display = "none";
}

if (token) {
    loginBtn.style.display = "none";
    registrationBtn.style.display = "none";
    profileBtn.style.display = "block";
    logOut.style.display = "block";
}
else {
    displayRegistrationButtons();
}

logOut.addEventListener('click', () => {
    localStorage.removeItem('token');
    localStorage.removeItem('refreshToken');
    displayRegistrationButtons();
})
```

### Пункт 10 - Додати оновлення профіля

#### JS частина

```
function updateProfile(data) {
    const id = data.id;

    fetch(`http://localhost:3000/profile/${id}`, {
        method: 'PATCH',
        headers: { 
            'Content-Type': 'application/json',
            'authorization': `Bearer ${localStorage.getItem('token')}`,
        },
        body: JSON.stringify({
           name: data.name,
        })
    })
    .then(response => {
        if (!response.ok) {
            return response.text().then(text => { throw new Error(text) });
        }
        return response.json();
    })
    .then(response => {
        inputName.setAttribute('disabled', 'true');
        saveButton.style.display = "none";
        editButton.style.display = "block";
        alert("Користувача оновлено");
    })
    .catch(er => console.log(er));
}

editButton.addEventListener('click', (event) => {
    event.preventDefault();

    inputName.removeAttribute('disabled');
    saveButton.style.display = "block";
    editButton.style.display = "none";
});

saveButton.addEventListener('click', (event) => {
    event.preventDefault();
    updateProfile({
        name: inputName.value,
        id: userId,
    });
});
```

#### Back частина

```
app.patch("/profile/:id", async (req, res) => {
  const authHeader = req.headers["authorization"]; // 🔒 Захищений маршрут
  const id = req.params.id; 
  const data = req.body;

  if (!authHeader) {
    return res.status(401).json({ message: "Немає токена" });
  }

  try {
    const token = authHeader.split(" ")[1];
    const decoded = jwt.verify(token, SECRET_KEY);
    const user = await User.findOne({ where: { id } });
    
    await User.update({ ...data }, {where:{id}});
    res.status(201).json({ message: "Користувача оновлено" });
  } catch (error) {
    res.status(401).json({ message: "Невірний токен" });
  }
});
```

### Пункт 11 - Зберігати користувачів у базі

```
const userExists = await User.findOne({ where: { email } });
if (userExists) return res.status(400).json({ message: "Користувач вже існує" });

// Хешування пароля
const hashedPassword = await bcrypt.hash(password, 10);
await User.create({ email, password: hashedPassword, name });
```

### Пункт 12 - реалізувати refresh token

#### JS частина

```
.catch(er => {
    console.log(er);
    refreshToken();
});

function refreshToken() {
    fetch("http://localhost:3000/refresh", {
        method: 'POST',
        headers: { 
            'Content-Type': 'application/json',
            'authorization': `Bearer ${localStorage.getItem('refreshToken')}`,
        },
    })
    .then(response => {
        if (!response.ok) {
            return response.text().then(text => { throw new Error(text) });
        }
        return response.json();
    })
    .then(response => {
        console.log(response);
        localStorage.setItem("token", response.token);
        getProfile();
    })
    .catch(er => {
        console.log(er);
    });
}
```

#### Back частина

```
app.post('/refresh', async (req, res) => {
    const refreshToken = req.headers["authorization"];
    if (!refreshToken) return res.sendStatus(401);

    const token = refreshToken.split(" ")[1];
    const decoded = jwt.verify(token, SECRET_KEY);
    jwt.verify(token, SECRET_KEY, (err, user) => {
        if (err) return res.sendStatus(401);
        const token = jwt.sign({ email: decoded.email, name: decoded.name, id: decoded.id }, SECRET_KEY, { expiresIn: "1m" });
        res.json({ token });
    });
});
```

### Пункт 14 - Обмежити кількість спроб входу

```
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 хвилин
    max: 2, // 5 спроб
    // Цей метод спрацює, коли ліміт буде вичерпано
    handler: (req, res, next, options) => {
    res.status(429).json({ 
        message: "Забагато невдалих спроб. Спробуйте через 15 хвилин." 
    });
    },
    standardHeaders: true, 
    legacyHeaders: false,
});

app.post('/login', loginLimiter, async (req, res) => {...}
```

<!-- ---

## Скріншоти

![Скрін 1](/assets/labs/lab-2/users.png)
![Скрін 2](/assets/labs/lab-2/posts.png)

--- -->

## Висновки
В цій лабораторній роботі я вивчив принципи побудови REST API, набув практичних навичок розробки серверного застосунку з використанням платформи Node.js і фреймворку Express, реалізував механізми реєстрації та авторизації користувачів, забезпечив валідацію вхідних даних, забезпечив обробку помилок, організував захищений доступ до ресурсів із використанням JWT-токенів і системи ролей користувачів.
