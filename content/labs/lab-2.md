## Тема, Мета, Місце розташування

**Тема:** Laba2

**Мета:** 
- Навчитися створювати базу даних у MySQL
- Освоїти виконання SQL-запитів (SELECT, INSERT, UPDATE, DELETE)
- Підключати серверну програму на Node.js до бази даних
- Використовувати ORM Sequelize для роботи з БД
- Реалізувати зв’язок One-to-Many між таблицями

**Місце розташування:**
- [GitHub](https://github.com/evitan22/IC-32_appRECORD-KravetsKyryl-FIOT-2026)
- [Live demo](https://evitan22.github.io/IC-32_appRECORD-KravetsKyryl-FIOT-2026/)

---

## Хід виконання

### Пункт 1

#### Моя БД 

```
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
    'MyDatabase', 
    'root',             
    '1234',   
    {
        host: 'localhost',
        dialect: 'mysql',
        logging: false,
    }
);

module.exports = sequelize;
```

#### Моя модель Post

```
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database'); 

const Post = sequelize.define('Post', {
    title: {
        type: DataTypes.STRING,
        allowNull: false,  
    },
    content: {
        type: DataTypes.STRING,
        allowNull: false,  
    },
});

module.exports = Post;
```

#### Моя модель User

```
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database'); // Імпортуємо підключення

const User = sequelize.define('User', {
    name: {
        type: DataTypes.STRING,
        allowNull: false
    },
    email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true
    }
});

module.exports = User;
```

### Пункт 2

```
const sequelize = require('./config/database');
const User = require('./models/User');
const Post = require('./models/Post');
User.hasMany(Post);
Post.belongsTo(User);

const mysql = require('mysql2/promise');

async function startApp() {
    try {
        await sequelize.authenticate();
        console.log('З’єднання з MySQL встановлено через mysql2!');

        await sequelize.sync({ alter: true }); 
        console.log('Таблиці в базі оновлено.');
    }...
```

### Пункт 3, 5, 6 - INSERT

#### Sequelize

```
await User.create({
    name: "Kyryl",
    email: "post@gmail.com"
});

await Post.create({
    title: 'My first post',
    content:'Hello world',
    UserId: 1
});
```

#### SQL

```
await connection.execute(
    'INSERT INTO Users (name, email, createdAt, updatedAt) VALUES (?, ?, NOW(), NOW())',
    ['Anastasia', 'nk@gmail.com']
);
```

### UPDATE

#### Sequelize

```
await Post.update(
    {title:'Updated post'},
    {where:{id:2}}
);
```

#### SQL

```
await connection.execute(
    'UPDATE Posts SET title = ?, content = ?, updatedAt = NOW() WHERE id = ?',
    ['SQL updated title', 'One more update', 3]
);
```

### SELECT

#### Sequelize

```
const post = await Post.findByPk(3);
console.log(post);
```

#### SQL

```
const [rows] = await connection.execute('SELECT * FROM users');
rows.forEach(element => console.log(element));
```

### DELETE

#### Sequelize

```
await Post.destroy({
    where:{id:1}
});
```

#### SQL

```
await connection.execute(
    'DELETE FROM Posts WHERE id = ?',
    [8]
);
```

---

## Скріншоти

![Скрін 1](/assets/labs/lab-2/users.png)
![Скрін 2](/assets/labs/lab-2/posts.png)

---

## Висновки
В цій лабораторній роботі я навчився створювати базу даних у MySQL, освоїв виконання SQL-запитів (SELECT, INSERT, UPDATE, DELETE), підключив серверну програму на Node.js до бази даних, використав ORM Sequelize для роботи з БД, реалізував зв’язок One-to-Many між таблицями
