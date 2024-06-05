# db-course-work

## Содержание

* [Предметная область](#description)
* [Создание таблиц](#create)
* [Заполнение таблиц](#insert)
* [Запросы](#queries)

## Предметная область <a name="description"></a>
В качестве предметной области выбрана тема проектирования базы данных для образовательной платформы. 

Основными элементами предметной области должны являться:
-  Пользователь
-  Роли пользователей
-  Курс
-  Материал курса (урок)
-  Задание для курса
-  Оценка выполненного задания

## Создание таблиц <a name="create"></a>

```SQL
CREATE TABLE IF NOT EXISTS roles (
	id SERIAL PRIMARY KEY,
	name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE IF NOT EXISTS users (
	id SERIAL PRIMARY KEY,
	username VARCHAR(50) NOT NULL UNIQUE,
	first_name VARCHAR(50) NOT NULL,
	last_name VARCHAR(50) NOT NULL,
	email VARCHAR(100) NOT NULL UNIQUE,
	hashed_password VARCHAR(255) NOT NULL,
	role_id INT NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	FOREIGN KEY (role_id) REFERENCES roles(id)
);

CREATE TABLE IF NOT EXISTS courses (
	id SERIAL PRIMARY KEY,
	name VARCHAR(100) NOT NULL UNIQUE,
	description TEXT,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS lessons (
	id SERIAL PRIMARY KEY,
	name VARCHAR(100) NOT NULL UNIQUE,
	description TEXT,
	text TEXT,
	course_id INT NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	FOREIGN KEY (course_id) REFERENCES courses(id)
);

CREATE TABLE IF NOT EXISTS tasks (
	id SERIAL PRIMARY KEY,
	name VARCHAR(100) NOT NULL UNIQUE,
	description TEXT,
	lesson_id INT NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	FOREIGN KEY (lesson_id) REFERENCES lessons(id)
);

CREATE TABLE IF NOT EXISTS task_answers (
	id SERIAL PRIMARY KEY,
	user_id INT NOT NULL,
	task_id INT NOT NULL,
	answer TEXT NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	FOREIGN KEY (user_id) REFERENCES users(id),
	FOREIGN KEY (task_id) REFERENCES tasks(id)
);

CREATE TABLE IF NOT EXISTS grades (
	id SERIAL PRIMARY KEY,
	task_answer_id INT NOT NULL,
	grade INT NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	FOREIGN KEY (task_answer_id) REFERENCES task_answers(id)
);

CREATE TABLE IF NOT EXISTS users_courses (
	user_id INT NOT NULL,
	course_id INT NOT NULL,
	PRIMARY KEY (user_id, course_id),
	FOREIGN KEY (user_id) REFERENCES users(id),
	FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

### Заполнение таблиц <a name="insert"></a>

```SQL
INSERT INTO ROLES (NAME)
VALUES
	('Студент'),
	('Преподаватель'),
RETURNING id;

INSERT INTO users (username, first_name, last_name, email, hashed_password, role_id) 
VALUES
	('ivan', 'Иван', 'Иванов', 'ivanov@example.com', 'hashed_password_example', 1),
	('petr', 'Петр', 'Петров', 'petrov@example.com', 'hashed_password_example', 1),
	('anna', 'Анна', 'Сидорова', 'sidorova@example.com', 'hashed_password_example', 1),
	('maria', 'Мария', 'Павлова', 'pavlova@example.com', 'hashed_password_example', 2),
	('aleksey', 'Алексей', 'Козлов', 'kozlov@example.com', 'hashed_password_example', 1)
RETURNING id;

INSERT INTO courses (name, description) 
VALUES
	('Базы данных', 'Курс по основам баз данных'),
	('Программирование на Python', 'Курс по языку программирования Python'),
	('Web-разработка', 'Курс по разработке веб-приложений'),
	('Машинное обучение', 'Курс по основам машинного обучения'),
	('Сетевая безопасность','Курс по защите компьютерных сетей')
RETURNING id;

INSERT INTO lessons (name, description, course_id) 
VALUES
	('Введение в SQL', 'Основные понятия и операции SQL. SQL (Structured Query Language) — язык структурированных запросов.', 1),
	('Работа с данными в Python', 'Использование библиотеки Pandas для анализа данных в Python.', 1),
	('Основы HTML и CSS', 'Изучение основных тегов HTML и каскадных таблиц стилей (CSS).', 1),
	('Введение в машинное обучение', 'Основные понятия и методы машинного обучения.', 2),
	('Защита сетей от атак', 'Основные принципы защиты компьютерных сетей от хакерских атак.', 3)
RETURNING id;

INSERT INTO tasks (name, description, lesson_id) 
VALUES
	('Задание 1', 'Первое задание по SQL', 1),
	('Задание 2', 'Второе задание по Python', 2),
	('Задание 3', 'Третье задание по HTML', 3),
	('Задание 4', 'Четвертое задание по машинному обучению', 4),
	('Задание 5', 'Пятое задание по защите сетей', 5)
RETURNING id;

INSERT INTO task_answers (user_id, task_id, answer)
VALUES 
	(1, 1, 'Ответ на первое задание'),
	(1, 2, 'Ответ на второе задание'),
	(1, 3, 'Ответ на третье задание'),
	(2, 4, 'Ответ на четвертое задание'),
	(2, 5, 'Ответ на пятое задание')
RETURNING id;

INSERT INTO grades (task_answer_id, grade) 
VALUES
	(1, 95),
	(2, 80),
	(3, 75),
	(4, 90),
	(5, 85)
RETURNING id;

INSERT INTO users_courses (user_id, course_id) VALUES
	(1, 1),
	(1, 2),
	(1, 3),
	(2, 3),
	(2, 4);
```


## Запросы <a name="queries"></a>

Запрос для выборки всех пользователей. 

```SQL
SELECT first_name, last_name 
FROM users;
```

Вывод средней оценки по всем пользователям и количество ответов на задания

```SQL
SELECT AVG(grade), COUNT(grade)
FROM grades;
```

Выборка имен и фамилий пользователей, которые прикрепили задание на задачу с определенным ID

```SQL
SELECT first_name, last_name, email
FROM users
WHERE id IN (
	SELECT user_id FROM task_answers WHERE task_id = 1
);
```

Запрос для вывода пользователя по его имени и названию роли. Используется подзапрос для получения ID роли по ее названию

```SQL
SELECT first_name, last_name, email 
FROM users 
WHERE first_name = 'Иван'
AND role_id IN (
	SELECT id FROM roles WHERE name = 'Студент'
);
```


Запрос для вывода средней оценка по выполненным заданиям пользователя по его имени

```SQL
SELECT AVG(grade) AS average_grade
FROM grades
WHERE task_answer_id IN (
	SELECT id FROM task_answers WHERE user_id IN (
		SELECT id from users WHERE first_name = 'Иван'
	)
);
```


Запрос для вставки новой записи в таблицу courses, возвращает ID новой записи

```SQL
INSERT INTO courses (name, description) 
VALUES
	('Разработка на Django', 'Курс по основам Django')
RETURNING id;
```


Запрос для изменения email пользователя, возвращает имя и фамилию пользователя и новый email

```SQL
UPDATE users
SET email = 'updated_email@example.com'
WHERE id IN (
	SELECT id FROM users WHERE first_name = 'Иван'
)
RETURNING first_name, last_name, email;
```
