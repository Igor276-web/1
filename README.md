Clash Royale - Веб-сайт на Python (Flask)

Вот полный код для создания веб-сайта про Clash Royale на Python с использованием фреймворка Flask:

1. Структура проекта

clash-royale-site/
│
├── app.py
├── requirements.txt
├── static/
│   ├── css/
│   │   └── style.css
│   └── images/
│       ├── logo.png
│       ├── cards/
│       └── arena.jpg
│
└── templates/
    ├── base.html
    ├── index.html
    ├── cards.html
    ├── decks.html
    └── about.html
2. Основное приложение Flask

app.py:

`python
from flask import Flask, render_template, request, jsonify
import json
import random
from datetime import datetime

app = Flask(name)

# Данные о картах Clash Royale (упрощенный вариант)
CARDS_DATA = [
    {"id": 1, "name": "Великан", "elixir": 5, "rarity": "Обычная", "type": "Войска", "arena": 0},
    {"id": 2, "name": "Волшебник", "elixir": 5, "rarity": "Обычная", "type": "Войска", "arena": 5},
    {"id": 3, "name": "Принц", "elixir": 5, "rarity": "Эпическая", "type": "Войска", "arena": 7},
    {"id": 4, "name": "Пека", "elixir": 4, "rarity": "Обычная", "type": "Войска", "arena": 1},
    {"id": 5, "name": "Мини-Пека", "elixir": 2, "rarity": "Обычная", "type": "Войска", "arena": 0},
    {"id": 6, "name": "Огненный дух", "elixir": 1, "rarity": "Обычная", "type": "Войска", "arena": 2},
    {"id": 7, "name": "Мега-Рыцарь", "elixir": 7, "rarity": "Легендарная", "type": "Войска", "arena": 10},
    {"id": 8, "name": "Принцесса", "elixir": 3, "rarity": "Легендарная", "type": "Войска", "arena": 7},
    {"id": 9, "name": "Ледяной Дух", "elixir": 1, "rarity": "Обычная", "type": "Войска", "arena": 8},
    {"id": 10, "name": "Огненный шар", "elixir": 4, "rarity": "Обычная", "type": "Заклинание", "arena": 0},
    {"id": 11, "name": "Стрелы", "elixir": 3, "rarity": "Обычная", "type": "Заклинание", "arena": 0},
    {"id": 12, "name": "Бочка с варварами", "elixir": 6, "rarity": "Эпическая", "type": "Заклинание", "arena": 3}
]

# Популярные колоды
DECKS_DATA = [
    {
        "name": "Голем-Луна",
        "cards": ["Голем", "Лунный Дух", "Ночная Ведьма", "Могила", "Мега-Минотавр", "Стрелы", "Ярость", "Тёмный Принц"],
        "elixir_cost": 4.1,
        "type": "Толкание",
        "win_rate": 58
    },
    {
        "name": "X-Bow 2.9",
        "cards": ["X-Bow", "Ледяной Дух", "Ледяной Голем", "Скелеты", "Арбалетчик", "Огненный шар", "Бревно", "Тесла"],
        "elixir_cost": 2.9,
        "type": "Контроль",
        "win_rate": 52
    },
    {
        "name": "Hog 2.6",
        "cards": ["Кабан", "Пушка", "Ледяной Дух", "Скелеты", "Огненный шар", "Бревно", "Мускетер", "Ледяной Голем"],
        "elixir_cost": 2.6,
        "type": "Цикл",
        "win_rate": 55
    },
    {
        "name": "Lavaloon",
        "cards": ["Лава-Гончая", "Воздушный шар", "Мега-Минотавр", "Охотник за скелетами", "Гоблины", "Стрелы", "Огненный шар", "Тёмный Принц"],
        "elixir_cost": 4.3,
        "type": "Воздушная атака",
        "win_rate": 57
    }
]

# Новости
NEWS_DATA = [
    {
        "title": "Новый сезон: Королевские легенды",
        "date": "2023-11-01",
        "content": "В новом сезоне представлены уникальные награды и новые карты!"
    },
    {
        "title": "Баланс обновления от 25 октября",
        "date": "2023-10-25",
        "content": "Изменения в характеристиках нескольких карт для улучшения игрового баланса."
    },
    {
        "title": "Турнир Clash Royale Championship 2023",
        "date": "2023-10-15",
        "content": "Регистрация на мировой чемпионат по Clash Royale открыта!"
    },
    {
        "title": "Новая арена: Легендарная долина",
        "date": "2023-10-05",
        "content": "Игроки могут разблокировать новую арену при достижении 5000 трофеев."
    }
]

@app.route('/')
def index():
    """Главная страница"""
    return render_template('index.html', 
                          cards=CARDS_DATA[:6], 
                          news=NEWS_DATA,
                          decks=DECKS_DATA)
