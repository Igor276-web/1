nuclear-physics-site/
│
├── app.py
├── requirements.txt
├── static/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── main.js
│   └── images/
│       └── (необязательно - будут использоваться внешние изображения)
│
└── templates/
    ├── base.html
    ├── index.html
    ├── atoms.html
    ├── radioactivity.html
    ├── reactions.html
    ├── calculator.html
    └── about.html
from flask import Flask, render_template, request, jsonify
import json
import math
import random
from datetime import datetime

app = Flask(name)

# Данные об элементах
ELEMENTS_DATA = [
    {"id": 1, "name": "Водород", "symbol": "H", "atomic_number": 1, "atomic_mass": 1.008, "type": "Неметалл"},
    {"id": 2, "name": "Гелий", "symbol": "He", "atomic_number": 2, "atomic_mass": 4.0026, "type": "Инертный газ"},
    {"id": 3, "name": "Литий", "symbol": "Li", "atomic_number": 3, "atomic_mass": 6.94, "type": "Щелочной металл"},
    {"id": 6, "name": "Углерод", "symbol": "C", "atomic_number": 6, "atomic_mass": 12.011, "type": "Неметалл"},
    {"id": 8, "name": "Кислород", "symbol": "O", "atomic_number": 8, "atomic_mass": 15.999, "type": "Неметалл"},
    {"id": 26, "name": "Железо", "symbol": "Fe", "atomic_number": 26, "atomic_mass": 55.845, "type": "Переходный металл"},
    {"id": 79, "name": "Золото", "symbol": "Au", "atomic_number": 79, "atomic_mass": 196.97, "type": "Переходный металл"},
    {"id": 92, "name": "Уран", "symbol": "U", "atomic_number": 92, "atomic_mass": 238.03, "type": "Актиноид"},
    {"id": 94, "name": "Плутоний", "symbol": "Pu", "atomic_number": 94, "atomic_mass": 244, "type": "Актиноид"},
]

# Данные о радиоактивных элементах
RADIOACTIVE_ELEMENTS = [
    {"name": "Уран-235", "half_life": "7.04e8 лет", "decay_type": "α-распад", "uses": "Ядерное топливо"},
    {"name": "Уран-238", "half_life": "4.47e9 лет", "decay_type": "α-распад", "uses": "Ядерное топливо"},
    {"name": "Плутоний-239", "half_life": "2.41e4 лет", "decay_type": "α-распад", "uses": "Ядерное оружие, топливо"},
    {"name": "Кобальт-60", "half_life": "5.27 лет", "decay_type": "β-распад", "uses": "Медицина (лучевая терапия)"},
    {"name": "Углерод-14", "half_life": "5730 лет", "decay_type": "β-распад", "uses": "Радиоуглеродное датирование"},
    {"name": "Радий-226", "half_life": "1600 лет", "decay_type": "α-распад", "uses": "Исторически - люминофоры"},
    {"name": "Торий-232", "half_life": "1.405e10 лет", "decay_type": "α-распад", "uses": "Потенциальное ядерное топливо"},
]

# Известные ядерные реакции
REACTIONS_DATA = [
    {"name": "Деление урана-235", "equation": "²³⁵U + n → ¹⁴¹Ba + ⁹²Kr + 3n", "energy": "~200 МэВ", "type": "Деление"},
    {"name": "Термоядерная реакция (Солнце)", "equation": "4¹H → ⁴He + 2e⁺ + 2νₑ + 2γ", "energy": "26.7 МэВ", "type": "Синтез"},
    {"name": "Альфа-распад", "equation": "²³⁸U → ²³⁴Th + ⁴He", "energy": "4.27 МэВ", "type": "Распад"},
    {"name": "Бета-распад", "equation": "¹⁴C → ¹⁴N + e⁻ + ν̄ₑ", "energy": "0.156 МэВ", "type": "Распад"},
    {"name": "Реакция дейтерий-тритий", "equation": "²H + ³H → ⁴He + n", "energy": "17.6 МэВ", "type": "Синтез"},
]

# Новости и открытия
NEWS_DATA = [
    {"title": "Управляемый термоядерный синтез: новый рекорд", "date": "2023-12-15", "content": "Ученые установили новый рекорд по удержанию плазмы в токамаке."},
    {"title": "Новый изотоп открыт в ЦЕРНе", "date": "2023-11-20", "content": "Международная группа исследователей открыла новый сверхтяжелый изотоп."},
    {"title": "Ядерные батареи для космических миссий", "date": "2023-10-10", "content": "Разработаны новые компактные ядерные источники энергии для дальних космических полетов."},
    {"title": "Лечение рака с помощью альфа-частиц", "date": "2023-09-05", "content": "Новый метод терапии использует альфа-излучение для точечного уничтожения раковых клеток."},
]

@app.route('/')
def index():
    """Главная страница"""
    return render_template('index.html', 
                          elements=ELEMENTS_DATA[:6], 
                          news=NEWS_DATA,
                          reactions=REACTIONS_DATA[:3])

@app.route('/atoms')
def atoms():
    """Страница о строении атома"""
    return render_template('atoms.html', elements=ELEMENTS_DATA)
    @app.route('/radioactivity')
def radioactivity():
    """Страница о радиоактивности"""
    return render_template('radioactivity.html', 
                          elements=RADIOACTIVE_ELEMENTS,
                          decay_types=["α-распад", "β-распад", "γ-излучение", "спонтанное деление"])

@app.route('/reactions')
def reactions():
    """Страница о ядерных реакциях"""
    type_filter = request.args.get('type')
    
    filtered_reactions = REACTIONS_DATA
    
    if type_filter and type_filter != 'all':
        filtered_reactions = [r for r in filtered_reactions if r['type'] == type_filter]
    
    types = list(set(r['type'] for r in REACTIONS_DATA))
    
    return render_template('reactions.html', 
                          reactions=filtered_reactions, 
                          types=types,
                          current_type=type_filter)

@app.route('/calculator')
def calculator():
    """Калькулятор ядерной физики"""
    return render_template('calculator.html')

@app.route('/about')
def about():
    """Страница о сайте"""
    return render_template('about.html')

# API эндпоинты
@app.route('/api/elements')
def api_elements():
    """API для получения элементов"""
    return jsonify(ELEMENTS_DATA)

@app.route('/api/calculate-energy', methods=['POST'])
def calculate_energy():
    """API для расчета энергии по формуле E=mc²"""
    data = request.get_json()
    
    try:
        mass = float(data.get('mass', 0))
        energy = mass * (299792458 ** 2)  # E = mc²
        
        # Конвертация в разные единицы
        joules = energy
        megajoules = joules / 1e6
        kilotons_tnt = joules / 4.184e12
        
        return jsonify({
            "success": True,
            "result": {
                "joules": joules,
                "megajoules": megajoules,
                "kilotons_tnt": kilotons_tnt,
                "formatted": f"{joules:.2e} Дж"
            }
        })
    except ValueError:
        return jsonify({"success": False, "error": "Некорректное значение массы"})

@app.route('/api/calculate-half-life', methods=['POST'])
def calculate_half_life():
    """API для расчета оставшегося количества радиоактивного вещества"""
    data = request.get_json()
    
    try:
        initial_amount = float(data.get('initial_amount', 1))
        half_life = float(data.get('half_life', 1))
        time_passed = float(data.get('time_passed', 1))
        
        # Формула: N = N0 * (1/2)^(t/T)
        remaining_amount = initial_amount * (0.5 ** (time_passed / half_life))
        decayed_amount = initial_amount - remaining_amount
        decay_percentage = (decayed_amount / initial_amount) * 100
        
        return jsonify({
            "success": True,
            "result": {
                "remaining": remaining_amount,
                "decayed": decayed_amount,
                "decay_percentage": decay_percentage,
                "formatted": f"{remaining_amount:.4f} из {initial_amount}"
            }
        })
    except ValueError:
        return jsonify({"success": False, "error": "Некорректные входные данные"})

@app.route('/api/calculate-binding-energy', methods=['POST'])
def calculate_binding_energy():
    """API для расчета энергии связи ядра"""
    data = request.get_json()
    
    try:
        protons = int(data.get('protons', 1))
        neutrons = int(data.get('neutrons', 0))
        
        # Упрощенный расчет (реальные расчеты сложнее)
        # Используем полуэмпирическую формулу Вайцзеккера
        A = protons + neutrons  # Массовое число
        
        if A == 0:
            return jsonify({"success": False, "error": "Ядро должно содержать нуклоны"})
        
        # Коэффициенты формулы (в МэВ)
        av = 15.8  # Объемный член
        as_ = 18.3  # Поверхностный член
        ac = 0.714  # Кулоновский член
        asym = 23.2  # Член асимметрии
        delta = 12.0  # Член спаривания
        
        # Расчет энергии связи
        volume_term = av * A
        surface_term = as_ * (A ** (2/3))
        coulomb_term = ac * protons * (protons - 1) / (A ** (1/3))
        asymmetry_term = asym * ((A - 2*protons) ** 2) / A
        
        # Член спаривания
        if protons % 2 == 0 and neutrons % 2 == 0:  # четное-четное
            pairing_term = delta / (A ** 0.5)
        elif protons % 2 == 1 and neutrons % 2 == 1:  # нечетное-нечетное
            pairing_term = -delta / (A ** 0.5)
        else:  # четное-нечетное или нечетное-четное
            pairing_term = 0
        
        binding_energy = volume_term - surface_term - coulomb_term - asymmetry_term + pairing_term
        
        # Удельная энергия связи
        specific_energy = binding_energy / A if A > 0 else 0
        
        return jsonify({
            "success": True,
            "result": {
                "binding_energy": binding_energy,
                "specific_energy": specific_energy,
                "mass_number": A,
                "protons": protons,
                "neutrons": neutrons,
                "formatted": f"{binding_energy:.2f} МэВ"
            }
        })
    except ValueError:
        return jsonify({"success": False, "error": "Некорректные входные данные"})

if name == 'main':
    app.run(debug=True, host='0.0.0.0', port=5000)
    asym = 23.2  # Член асимметрии
        delta = 12.0  # Член спаривания
        
        # Расчет энергии связи
        volume_term = av * A
        surface_term = as_ * (A ** (2/3))
        coulomb_term = ac * protons * (protons - 1) / (A ** (1/3))
        asymmetry_term = asym * ((A - 2*protons) ** 2) / A
        
        # Член спаривания
        if protons % 2 == 0 and neutrons % 2 == 0:  # четное-четное
            pairing_term = delta / (A ** 0.5)
        elif protons % 2 == 1 and neutrons % 2 == 1:  # нечетное-нечетное
            pairing_term = -delta / (A ** 0.5)
        else:  # четное-нечетное или нечетное-четное
            pairing_term = 0
        
        binding_energy = volume_term - surface_term - coulomb_term - asymmetry_term + pairing_term
        
        # Удельная энергия связи
        specific_energy = binding_energy / A if A > 0 else 0
        
        return jsonify({
            "success": True,
            "result": {
                "binding_energy": binding_energy,
                "specific_energy": specific_energy,
                "mass_number": A,
                "protons": protons,
                "neutrons": neutrons,
                "formatted": f"{binding_energy:.2f} МэВ"
            }
        })
    except ValueError:
        return jsonify({"success": False, "error": "Некорректные входные данные"})

if name == 'main':
    app.run(debug=True, host='0.0.0.0', port=5000)
    Flask==2.3.3
    <!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Ядерная физика{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&family=Source+Code+Pro:wght@400;600&display=swap" rel="stylesheet">
</head>
<body>
    <!-- Навигация -->
    <nav class="navbar">
        <div class="container">
            <a href="{{ url_for('index') }}" class="logo">
                <i class="fas fa-atom"></i> Ядерная физика
            </a>
            <div class="nav-links">
                <a href="{{ url_for('index') }}" class="nav-link">Главная</a>
                <a href="{{ url_for('atoms') }}" class="nav-link">Строение атома</a>
                <a href="{{ url_for('radioactivity') }}" class="nav-link">Радиоактивность</a>
                <a href="{{ url_for('reactions') }}" class="nav-link">Ядерные реакции</a>
                <a href="{{ url_for('calculator') }}" class="nav-link">Калькулятор</a>
                <a href="{{ url_for('about') }}" class="nav-link">О проекте</a>
            </div>
            <button class="menu-toggle" onclick="toggleMenu()">
                <i class="fas fa-bars"></i>
            </button>
        </div>
    </nav>

    <!-- Основное содержимое -->
    <main class="main-content">
        {% block content %}{% endblock %}
    </main>

    <!-- Футер -->
    <footer class="footer">
        <div class="container">
            <div class="footer-content">
                <div class="footer-section">
                    <h3><i class="fas fa-atom"></i> Ядерная физика</h3>
                    <p>Образовательный ресурс о строении атома, радиоактивности и ядерных реакциях.</p>
                </div>
                <div class="footer-section">
                    <h3>Разделы</h3>
                    <a href="{{ url_for('atoms') }}">Строение атома</a>
                    <a href="{{ url_for('radioactivity') }}">Радиоактивность</a>
                    <a href="{{ url_for('reactions') }}">Ядерные реакции</a>
                    <a href="{{ url_for('calculator') }}">Калькуляторы</a>
                </div>
                <div class="footer-section">
                    <h3>Контакты</h3>
                    <p>Образовательный проект. Для связи: nuclear-physics@example.com</p>
                </div>
            </div>
            <div class="footer-bottom">
                <p>&copy; 2023 Ядерная физика. Образовательный проект.</p>
            </div>
        </div>
    </footer>

    <script src="{{ url_for('static', filename='js/main.js') }}"></script>
    {% block scripts %}{% endblock %}
</body>
</html>
{% extends "base.html" %}

{% block title %}Главная - Ядерная физика{% endblock %}

{% block content %}
<div class="hero">
    <div class="container">
        <h1>Ядерная физика</h1>
        <p>Изучение структуры атомного ядра, радиоактивности и ядерных реакций</p>
        <a href="{{ url_for('atoms') }}" class="btn btn-primary">Начать изучение</a>
    </div>
</div>

<div class="container">
    <!-- Введение -->
    <section class="section">
        <h2 class="section-title">Что такое ядерная физика?</h2>
        <div class="intro-content">
            <div class="intro-text">
                <p>Ядерная физика — раздел физики, изучающий структуру и свойства атомных ядер, а также их превращения — радиоактивные распады и ядерные реакции.</p>
                <p>Эта область науки имеет важное практическое значение для энергетики, медицины, археологии и многих других сфер человеческой деятельности.</p>
                <div class="key-concepts">
                    <div class="concept">
                        <i class="fas fa-bolt"></i>
                        <h3>Ядерная энергия</h3>
                        <p>Энергия, выделяющаяся при делении или синтезе атомных ядер</p>
                    </div>
                    <div class="concept">
                        <i class="fas fa-radiation"></i>
                        <h3>Радиоактивность</h3>
                        <p>Самопроизвольное превращение нестабильных атомных ядер</p>
                    </div>
                    <div class="concept">
                        <i class="fas fa-microscope"></i>
                        <h3>Строение ядра</h3>
                        <p>Изучение протонов, нейтронов и их взаимодействий</p>
                    </div>
                </div>
            </div>
            <div class="intro-image">
                <div class="atom-model">
                    <div class="nucleus">
                        <div class="proton"></div>
                        <div class="neutron"></div>
                    </div>
                    <div class="electron e1"></div>
                    <div class="electron e2"></div>
                    <div class="electron e3"></div>
                    <div class="orbit o1"></div>
                    <div class="orbit o2"></div>
                </div>
            </div>
        </div>
    </section>

    <!-- Основные элементы -->
    <section class="section">
        <h2 class="section-title">Ключевые элементы</h2>
        <div class="elements-grid">
            {% for element in elements %}
            <div class="element-card">
                <div class="element-header">
                    <div class="element-symbol">{{ element.symbol }}</div>
                    <div class="element-number">{{ element.atomic_number }}</div>
                </div>
                <div class="element-name">{{ element.name }}</div>
                <div class="element-mass">{{ element.atomic_mass }}</div>
                <div class="element-type">{{ element.type }}</div>
            </div>
            {% endfor %}
        </div>
        <div class="text-center">
            <a href="{{ url_for('atoms') }}" class="btn btn-secondary">Все элементы</a>
        </div>
    </section>

    <!-- Ядерные реакции -->
    <section class="section">
        <h2 class="section-title">Основные ядерные реакции</h2>
        <div class="reactions-grid">
            {% for reaction in reactions %}
            <div class="reaction-card">
                <div class="reaction-type {{ reaction.type|lower }}">{{ reaction.type }}</div>
                <h3>{{ reaction.name }}</h3>
                <div class="reaction-equation">{{ reaction.equation }}</div>
                <div class="reaction-energy">
                    <i class="fas fa-bolt"></i> {{ reaction.energy }}
                </div>
            </div>
            {% endfor %}
        </div>
        <div class="text-center">
            <a href="{{ url_for('reactions') }}" class="btn btn-secondary">Все реакции</a>
        </div>
    </section>
    <!-- Новости -->
    <section class="section">
        <h2 class="section-title">Последние новости науки</h2>
        <div class="news-grid">
            {% for item in news %}
            <div class="news-card">
                <div class="news-date">{{ item.date }}</div>
                <h3>{{ item.title }}</h3>
                <p>{{ item.content }}</p>
            </div>
            {% endfor %}
        </div>
    </section>

    <!-- Формула -->
    <section class="section formula-section">
        <div class="formula-box">
            <h2><i class="fas fa-square-root-alt"></i> Знаменитая формула</h2>
            <div class="formula">E = mc²</div>
            <p>Эквивалентность массы и энергии — основа ядерной энергетики</p>
            <a href="{{ url_for('calculator') }}" class="btn btn-primary">
                <i class="fas fa-calculator"></i> Калькулятор энергии
            </a>
        </div>
    </section>
</div>
{% endblock %}
{% extends "base.html" %}

{% block title %}Строение атома - Ядерная физика{% endblock %}

{% block content %}
<div class="container">
    <h1 class="section-title">Строение атома</h1>
    
    <div class="atom-structure">
        <div class="structure-info">
            <h2>Состав атомного ядра</h2>
            <p>Атом состоит из ядра и электронной оболочки. Ядро содержит:</p>
            <ul class="particle-list">
                <li>
                    <i class="fas fa-plus-circle proton-icon"></i>
                    <strong>Протоны</strong> — положительно заряженные частицы
                </li>
                <li>
                    <i class="fas fa-circle neutron-icon"></i>
                    <strong>Нейтроны</strong> — нейтральные частицы
                </li>
                <li>
                    <i class="fas fa-minus-circle electron-icon"></i>
                    <strong>Электроны</strong> — отрицательно заряженные частицы на орбиталях
                </li>
            </ul>
            
            <div class="structure-details">
                <div class="detail">
                    <div class="detail-title">Заряд ядра</div>
                    <div class="detail-value">+Z (равен числу протонов)</div>
                </div>
                <div class="detail">
                    <div class="detail-title">Массовое число</div>
                    <div class="detail-value">A = Z + N</div>
                </div>
                <div class="detail">
                    <div class="detail-title">Изотопы</div>
                    <div class="detail-value">Атомы с одинаковым Z, но разным N</div>
                </div>
            </div>
        </div>
        
        <div class="atom-model-large">
            <div class="nucleus-large">
                <div class="nucleus-label">Ядро<br>Протоны + Нейтроны</div>
                <div class="protons-neutrons">
                    <div class="proton-small"></div>
                    <div class="neutron-small"></div>
                    <div class="proton-small"></div>
                    <div class="neutron-small"></div>
                </div>
            </div>
            <div class="orbits-large">
                <div class="orbit-large o1">
                    <div class="electron-large e1"></div>
                </div>
                <div class="orbit-large o2">
                    <div class="electron-large e2"></div>
                    <div class="electron-large e3"></div>
                </div>
                <div class="orbit-large o3">
                    <div class="electron-large e4"></div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Таблица элементов -->
    <section class="section">
        <h2 class="section-title">Химические элементы</h2>
        <div class="elements-table-container">
            <table class="elements-table">
                <thead>
                    <tr>
                        <th>Символ</th>
                        <th>Название</th>
                        <th>Атомный номер</th>
                        <th>Атомная масса</th>
                        <th>Тип</th>
                    </tr>
                </thead>
                <tbody>
                    {% for element in elements %}
                    <tr>
                        <td class="element-symbol-cell">{{ element.symbol }}</td>
                        <td>{{ element.name }}</td>
                        <td>{{ element.atomic_number }}</td>
                        <td>{{ element.atomic_mass }}</td>
                        <td><span class="element-type-badge">{{ element.type }}</span></td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
    </section>
    
    <!-- Типы распадов -->
    <section class="section">
        <h2 class="section-title">Строение атомного ядра</h2>
        <div class="nucleus-structure">
            <div class="structure-card">
                <h3><i class="fas fa-magnet"></i> Сильное взаимодействие</h3>
                <p>Удерживает протоны и нейтроны в ядре, несмотря на кулоновское отталкивание протонов.</p>
                <div class="structure-detail">Действует на расстоянии ~10⁻¹⁵ м</div>
            </div>
            <div class="structure-card">
                <h3><i class="fas fa-balance-scale"></i> Соотношение частиц</h3>
                <p>В стабильных ядрах обычно N ≥ Z. С ростом атомного числа требуется больше нейтронов для стабильности.</p>
                <div class="structure-detail">Линия стабильности</div>
            </div>
            <div class="structure-card">
                <h3><i class="fas fa-chart-line"></i> Энергия связи</h3>
                <p>Энергия, необходимая для разделения ядра на отдельные нуклоны. Максимальна у железа-56.</p>
                <div class="structure-detail">~8.8 МэВ/нуклон для Fe-56</div>
            </div>
        </div>
    </section>
</div>

<style>
.atom-structure {
    display: flex;
    flex-wrap: wrap;
    gap: 40px;
    margin-bottom: 50px;
    align-items: center;
}

.structure-info {
    flex: 1;
    min-width: 300px;
}

.atom-model-large {
    flex: 1;
    min-width: 300px;
    height: 400px;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
}

.particle-list {
    list-style: none;
    padding: 0;
    margin: 20px 0;
}

.particle-list li {
    display: flex;
    align-items: center;
    margin-bottom: 15px;
    font-size: 18px;
}

.particle-list i {
    margin-right: 15px;
    font-size: 24px;
}

.proton-icon { color: #ff4757; }
.neutron-icon { color: #70a1ff; }
.electron-icon { color: #2ed573; }

.structure-details {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin-top: 30px;
}

.detail {
    background: rgba(255, 255, 255, 0.05);
    padding: 15px;
    border-radius: 10px;
    border-left: 4px solid #6c5ce7;
    flex: 1;
    min-width: 200px;
}

.detail-title {
    font-weight: bold;
    color: #6c5ce7;
    margin-bottom: 5px;
}

.detail-value {
    font-family: 'Source Code Pro', monospace;
    font-size: 18px;
}

.nucleus-large {
    width: 150px;
    height: 150px;
    background: radial-gradient(circle, #ff6b6b, #ff4757);
    border-radius: 50%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
    z-index: 2;
    box-shadow: 0 0 30px rgba(255, 71, 87, 0.5);
}

.nucleus-label {
    text-align: center;
    color: white;
    font-weight: bold;
    padding: 10px;
    font-size: 14px;
}

.protons-neutrons {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 5px;
    margin-top: 10px;
}

.proton-small, .neutron-small {
    width: 15px;
    height: 15px;
    border-radius: 50%;
}

.proton-small {
    background-color: white;
    border: 2px solid #ff4757;
}

.neutron-small {
    background-color: #70a1ff;
    border: 2px solid #1e90ff;
}

.orbits-large {
    position: absolute;
    width: 100%;
    height: 100%;
}

.orbit-large {
    position: absolute;
    border: 2px dashed rgba(255, 255, 255, 0.3);
    border-radius: 50%;
}

.orbit-large.o1 {
    width: 200px;
    height: 200px;
    top: calc(50% - 100px);
    left: calc(50% - 100px);
}

.orbit-large.o2 {
    width: 300px;
    height: 300px;
    top: calc(50% - 150px);
    left: calc(50% - 150px);
}

.orbit-large.o3 {
    width: 400px;
    height: 400px;
    top: calc(50% - 200px);
    left: calc(50% - 200px);
}

.electron-large {
    position: absolute;
    width: 20px;
    height: 20px;
    background: radial-gradient(circle, #7bed9f, #2ed573);
    border-radius: 50%;
    box-shadow: 0 0 10px #2ed573;
}
.electron-large.e1 { top: 10px; left: 50%; transform: translateX(-50%); }
.electron-large.e2 { top: 50%; right: 10px; transform: translateY(-50%); }
.electron-large.e3 { bottom: 10px; left: 50%; transform: translateX(-50%); }
.electron-large.e4 { top: 50%; left: 10px; transform: translateY(-50%); }

.elements-table-container {
    overflow-x: auto;
    margin-top: 20px;
}

.elements-table {
    width: 100%;
    border-collapse: collapse;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 10px;
    overflow: hidden;
}

.elements-table th {
    background: rgba(108, 92, 231, 0.3);
    padding: 15px;
    text-align: left;
    font-weight: 600;
}

.elements-table td {
    padding: 15px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.element-symbol-cell {
    font-weight: bold;
    color: #6c5ce7;
    font-size: 20px;
}

.element-type-badge {
    background: rgba(108, 92, 231, 0.2);
    padding: 5px 10px;
    border-radius: 20px;
    font-size: 14px;
    border: 1px solid #6c5ce7;
}

.nucleus-structure {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 25px;
    margin-top: 30px;
}

.structure-card {
    background: rgba(255, 255, 255, 0.05);
    padding: 25px;
    border-radius: 10px;
    border-top: 4px solid #6c5ce7;
}

.structure-card h3 {
    display: flex;
    align-items: center;
    margin-bottom: 15px;
    color: #6c5ce7;
}

.structure-card h3 i {
    margin-right: 10px;
}

.structure-card .structure-detail {
    margin-top: 15px;
    padding: 10px;
    background: rgba(108, 92, 231, 0.1);
    border-radius: 6px;
    font-family: 'Source Code Pro', monospace;
    text-align: center;
}

@media (max-width: 768px) {
    .atom-structure {
        flex-direction: column;
    }
    
    .atom-model-large {
        height: 300px;
    }
    
    .orbit-large.o1 { width: 150px; height: 150px; }
    .orbit-large.o2 { width: 225px; height: 225px; }
    .orbit-large.o3 { width: 300px; height: 300px; }
}
</style>
{% endblock %}
{% extends "base.html" %}

{% block title %}Радиоактивность - Ядерная физика{% endblock %}

{% block content %}
<div class="container">
    <h1 class="section-title">Радиоактивность</h1>
    
    <div class="radioactivity-intro">
        <div class="intro-text">
            <h2>Что такое радиоактивность?</h2>
            <p>Радиоактивность — самопроизвольное превращение нестабильных атомных ядер в другие ядра с испусканием различных частиц.</p>
            <p>Открыта в 1896 году Анри Беккерелем. Различают естественную и искусственную радиоактивность.</p>
        </div>
        <div class="radioactivity-types">
            <div class="type-item alpha">
                <div class="type-icon">α</div>
                <div class="type-info">
                    <h3>Альфа-распад</h3>
                    <p>Испускание ядра гелия-4 (2 протона + 2 нейтрона)</p>
                </div>
            </div>
            <div class="type-item beta">
                <div class="type-icon">β</div>
                <div class="type-info">
                    <h3>Бета-распад</h3>
                    <p>Превращение нейтрона в протон или протона в нейтрон</p>
                </div>
            </div>
            <div class="type-item gamma">
                <div class="type-icon">γ</div>
                <div class="type-info">
                    <h3>Гамма-излучение</h3>
                    <p>Испускание фотонов высоких энергий</p>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Виды распадов -->
    <section class="section">
        <h2 class="section-title">Типы радиоактивного распада</h2>
        <div class="decay-types">
            {% for decay_type in decay_types %}
            <div class="decay-card">
                <h3>{{ decay_type }}</h3>
                <div class="decay-equation">
                    {% if decay_type == "α-распад" %}
                    <div class="equation">²³⁸U → ²³⁴Th + α</div>
                    <div class="equation-desc">Ядро испускает α-частицу (ядро He)</div>
                    {% elif decay_type == "β-распад" %}
                    <div class="equation">¹⁴C → ¹⁴N + e⁻ + ν̄</div>
                    <div class="equation-desc">Нейтрон превращается в протон</div>
                    {% elif decay_type == "γ-излучение" %}
                    <div class="equation">⁶⁰Co* → ⁶⁰Co + γ</div>
                    <div class="equation-desc">Испускание фотона возбужденным ядром</div>
                    {% else %}
                    <div class="equation">²⁵⁶Fm → ² ядра-осколка</div>
                    <div class="equation-desc">Деление тяжелого ядра на два осколка</div>
                    {% endif %}
                </div>
                <div class="decay-properties">
                    {% if decay_type == "α-распад" %}
                    <div class="property"><strong>Проникающая способность:</strong> Низкая</div>
                    <div class="property"><strong>Ионизирующая способность:</strong> Высокая</div>
                    {% elif decay_type == "β-распад" %}
                    <div class="property"><strong>Проникающая способность:</strong> Средняя</div>
                    <div class="property"><strong>Ионизирующая способность:</strong> Средняя</div>
                    {% elif decay_type == "γ-излучение" %}
                    <div class="property"><strong>Проникающая способность:</strong> Высокая</div>
                    <div class="property"><strong>Ионизирующая способность:</strong> Низкая</div>
                    {% else %}
                    <div class="property"><strong>Проникающая способность:</strong> Осколки - низкая</div>
                    <div class="property"><strong>Выделяемая энергия:</strong> Очень высокая</div>
                    {% endif %}
                </div>
                </div>
            {% endfor %}
        </div>
    </section>
    
    <!-- Радиоактивные элементы -->
    <section class="section">
        <h2 class="section-title">Радиоактивные элементы</h2>
        <div class="radioactive-elements">
            {% for element in elements %}
            <div class="radioactive-element">
                <div class="element-name">{{ element.name }}</div>
                <div class="element-properties">
                    <div class="property">
                        <span class="property-label">Период полураспада:</span>
                        <span class="property-value">{{ element.half_life }}</span>
                    </div>
                    <div class="property">
                        <span class="property-label">Тип распада:</span>
                        <span class="property-value">{{ element.decay_type }}</span>
                    </div>
                    <div class="property">
                        <span class="property-label">Применение:</span>
                        <span class="property-value">{{ element.uses }}</span>
                    </div>
                </div>
            </div>
            {% endfor %}
        </div>
    </section>
    
    <!-- Период полураспада -->
    <section class="section">
        <div class="half-life-section">
            <div class="half-life-info">
                <h2><i class="fas fa-hourglass-half"></i> Период полураспада</h2>
                <p>Время, за которое количество радиоактивного вещества уменьшается вдвое.</p>
                <div class="formula-box">
                    <div class="formula">N(t) = N₀ × (½)<sup>t/T</sup></div>
                    <p>где N₀ — начальное количество, t — время, T — период полураспада</p>
                </div>
                <a href="{{ url_for('calculator') }}" class="btn btn-primary">
                    <i class="fas fa-calculator"></i> Рассчитать распад
                </a>
            </div>
            <div class="half-life-visual">
                <div class="decay-graph">
                    <div class="graph-point" style="left: 0%; top: 0%">100%</div>
                    <div class="graph-point" style="left: 25%; top: 50%">50%</div>
                    <div class="graph-point" style="left: 50%; top: 25%">25%</div>
                    <div class="graph-point" style="left: 75%; top: 12.5%">12.5%</div>
                    <div class="graph-point" style="left: 100%; top: 6.25%">6.25%</div>
                    <div class="graph-line"></div>
                </div>
                <div class="graph-labels">
                    <div class="label">0T</div>
                    <div class="label">1T</div>
                    <div class="label">2T</div>
                    <div class="label">3T</div>
                    <div class="label">4T</div>
                </div>
            </div>
        </div>
    </section>
</div>

<style>
.radioactivity-intro {
    margin-bottom: 50px;
}

.intro-text {
    margin-bottom: 40px;
}

.radioactivity-types {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}

.type-item {
    flex: 1;
    min-width: 250px;
    display: flex;
    align-items: center;
    background: rgba(255, 255, 255, 0.05);
    padding: 20px;
    border-radius: 10px;
    border-left: 4px solid;
}

.type-item.alpha { border-color: #ff4757; }
.type-item.beta { border-color: #2ed573; }
.type-item.gamma { border-color: #ffa502; }

.type-icon {
    font-size: 40px;
    font-weight: bold;
    margin-right: 20px;
    width: 60px;
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
}

.type-item.alpha .type-icon {
    background: rgba(255, 71, 87, 0.2);
    color: #ff4757;
}

.type-item.beta .type-icon {
    background: rgba(46, 213, 115, 0.2);
    color: #2ed573;
}

.type-item.gamma .type-icon {
    background: rgba(255, 165, 2, 0.2);
    color: #ffa502;
}

.decay-types {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 25px;
    margin-bottom: 40px;
}
.decay-card {
    background: rgba(255, 255, 255, 0.05);
    padding: 25px;
    border-radius: 10px;
    border-top: 4px solid #6c5ce7;
}

.decay-equation {
    margin: 20px 0;
    padding: 15px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 8px;
    font-family: 'Source Code Pro', monospace;
    font-size: 18px;
    text-align: center;
}

.equation-desc {
    font-size: 14px;
    color: #aaa;
    margin-top: 10px;
    font-family: 'Roboto', sans-serif;
}

.decay-properties {
    margin-top: 20px;
}

.property {
    margin-bottom: 10px;
    padding: 8px;
    background: rgba(255, 255, 255, 0.02);
    border-radius: 5px;
}

.radioactive-elements {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 20px;
    margin-top: 30px;
}

.radioactive-element {
    background: rgba(255, 255, 255, 0.05);
    padding: 20px;
    border-radius: 10px;
    border-left: 4px solid #ff4757;
}

.element-name {
    font-size: 22px;
    font-weight: bold;
    color: #ff4757;
    margin-bottom: 15px;
}

.element-properties {
    font-size: 15px;
}

.property {
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
}

.property-label {
    color: #aaa;
}

.property-value {
    font-weight: 500;
    text-align: right;
}

.half-life-section {
    display: flex;
    flex-wrap: wrap;
    gap: 40px;
    align-items: center;
    background: rgba(108, 92, 231, 0.1);
    padding: 30px;
    border-radius: 15px;
    margin-top: 30px;
}

.half-life-info {
    flex: 1;
    min-width: 300px;
}

.half-life-visual {
    flex: 1;
    min-width: 300px;
}

.formula-box {
    margin: 25px 0;
    padding: 20px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 10px;
    text-align: center;
}

.formula-box .formula {
    font-size: 32px;
    font-family: 'Source Code Pro', monospace;
    margin-bottom: 10px;
}

.decay-graph {
    position: relative;
    height: 200px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 10px;
    margin-top: 20px;
}

.graph-point {
    position: absolute;
    width: 20px;
    height: 20px;
    background: #ff4757;
    border-radius: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-size: 12px;
}

.graph-line {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.graph-line::after {
    content: '';
    position: absolute;
    top: 0;
    left: 10%;
    width: 80%;
    height: 100%;
    border-bottom: 2px dashed #6c5ce7;
    border-left: 2px dashed #6c5ce7;
}

.graph-labels {
    display: flex;
    justify-content: space-between;
    padding: 10px 10% 0;
}

.graph-labels .label {
    font-size: 14px;
    color: #aaa;
}

@media (max-width: 768px) {
    .radioactivity-types {
        flex-direction: column;
    }
    
    .half-life-section {
        flex-direction: column;
    }
}
</style>
{% endblock %}
{% extends "base.html" %}

{% block title %}Ядерные реакции - Ядерная физика{% endblock %}

{% block content %}
<div class="container">
    <h1 class="section-title">Ядерные реакции</h1>
    
    <div class="reactions-intro">
        <p>Ядерные реакции — процессы превращения атомных ядер при взаимодействии с другими ядрами или элементарными частицами.</p>
        
        <div class="reaction-filter">
            <label>Фильтр по типу реакции:</label>
            <select class="filter-select" id="typeFilter" onchange="filterReactions()">
                <option value="all" {% if not current_type %}selected{% endif %}>Все типы</option>
                {% for type in types %}
                <option value="{{ type }}" {% if current_type == type %}selected{% endif %}>{{ type }}</option>
                {% endfor %}
            </select>
        </div>
    </div>
    
    <!-- Реакции -->
    <div class="reactions-list">
        {% for reaction in reactions %}
        <div class="reaction-item">
            <div class="reaction-header">
                <h2>{{ reaction.name }}</h2>
                <span class="reaction-type {{ reaction.type|lower }}">{{ reaction.type }}</span>
            </div>
            
            <div class="reaction-equation">
                {{ reaction.equation }}
            </div>
            
            <div class="reaction-details">
                <div class="detail">
                    <i class="fas fa-bolt"></i>
                    <span class="detail-label">Энергия:</span>
                    <span class="detail-value">{{ reaction.energy }}</span>
                </div>
                
                <div class="detail">
                    <i class="fas fa-info-circle"></i>
                    <span class="detail-label">Описание:</span>
                    <span class="detail-value">
                        {% if reaction.type == "Деление" %}
                        Тяжелое ядро распадается на два более легких ядра с выделением энергии
                        {% elif reaction.type == "Синтез" %}
                        Легкие ядра объединяются в более тяжелое ядро с выделением энергии
                        {% else %}
                        Самопроизвольное превращение нестабильного ядра
                        {% endif %}
                    </span>
                </div>
            </div>
            
            {% if reaction.type == "Деление" %}
            <div class="reaction-example">
                <h3><i class="fas fa-atom"></i> Пример цепной реакции</h3>
                <p>Нейтрон попадает в ядро урана-235, вызывая его деление с выделением энергии и новых нейтронов, которые могут вызвать деление других ядер.</p>
                <div class="chain-reaction">
                    <div class="nucleus">²³⁵U</div>
                    <div class="arrow">+ n →</div>
                    <div class="fission">
                        <div class="fragment">¹⁴¹Ba</div>
                        <div class="plus">+</div>
                        <div class="fragment">⁹²Kr</div>
                        <div class="plus">+</div>
                        <div class="neutrons">3n</div>
                    </div>
                </div>
            </div>
            {% elif reaction.type == "Синтез" %}
            <div class="reaction-example">
                <h3><i class="fas fa-sun"></i> Термоядерная реакция</h3>
                <p>Происходит в недрах звезд, включая Солнце. Требует высоких температур (миллионы градусов) для преодоления кулоновского барьера.</p>
            </div>
            {% endif %}
        </div>
        {% endfor %}
    </div>
    
    <!-- Энергия связи -->
    <section class="section">
        <div class="binding-energy">
            <h2 class="section-title">Энергия связи ядра</h2>
            <p>Энергия, которую необходимо затратить, чтобы разделить ядро на отдельные нуклоны.</p>
            
            <div class="energy-curve">
                <div class="curve-point" style="left: 10%; bottom: 30%">
                    <div class="point-label">H</div>
                    <div class="point-value">1.1 МэВ/нуклон</div>
                </div>
                <div class="curve-point" style="left: 30%; bottom: 70%">
                    <div class="point-label">He</div>
                    <div class="point-value">7.1 МэВ/нуклон</div>
                </div>
                <div class="curve-point" style="left: 50%; bottom: 85%">
                    <div class="point-label">Fe</div>
                    <div class="point-value">8.8 МэВ/нуклон</div>
                </div>
                <div class="curve-point" style="left: 70%; bottom: 75%">
                    <div class="point-label">U</div>
                    <div class="point-value">7.6 МэВ/нуклон</div>
                </div>
                <div class="curve-line"></div>
            </div>
            
            <div class="curve-explanation">
                <div class="explanation-item">
                    <div class="exp-number">1</div>
                    <div class="exp-text">Энергия связи на нуклон быстро растет для легких ядер</div>
                </div>
                <div class="explanation-item">
                    <div class="exp-number">2</div>
                    <div class="exp-text">Максимум у железа-56 (~8.8 МэВ/нуклон)</div>
                </div>
                <div class="explanation-item">
                    <div class="exp-number">3</div>
                    <div class="exp-text">Для тяжелых ядер энергия связи уменьшается</div>
                </div>
            </div>
            
            <div class="energy-applications">
                <h3><i class="fas fa-lightbulb"></i> Практическое применение</h3>
                <div class="applications-grid">
                    <div class="application">
                        <h4>Деление тяжелых ядер</h4>
                        <p>Ядра тяжелее железа выделяют энергию при делении (ядерные реакторы)</p>
                    </div>
                    <div class="application">
                        <h4>Синтез легких ядер</h4>
                        <p>Ядра легче железа выделяют энергию при синтезе (термоядерные реакции)</p>
                    </div>
                </div>
            </div>
        </div>
    </section>
</div>

<script>
function filterReactions() {
    const type = document.getElementById('typeFilter').value;
    window.location.href = type === 'all' ? '/reactions' : /reactions?type=${type};
}
</script>

<style>
.reactions-intro {
    margin-bottom: 40px;
}

.reaction-filter {
    margin-top: 20px;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 15px;
}

.reaction-filter label {
    font-weight: 500;
}

.filter-select {
    background: rgba(255, 255, 255, 0.1);
    color: white;
    border: 1px solid #6c5ce7;
    padding: 10px 15px;
    border-radius: 8px;
    font-size: 16px;
    min-width: 200px;
}

.reactions-list {
    margin-bottom: 50px;
}

.reaction-item {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15px;
    padding: 30px;
    margin-bottom: 30px;
    border-left: 5px solid #6c5ce7;
}

.reaction-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 25px;
    flex-wrap: wrap;
    gap: 15px;
}

.reaction-header h2 {
    margin: 0;
    color: #6c5ce7;
}

.reaction-type {
    padding: 8px 20px;
    border-radius: 20px;
    font-weight: 600;
    font-size: 14px;
}

.reaction-type.деление {
    background: rgba(255, 71, 87, 0.2);
    color: #ff4757;
    border: 1px solid #ff4757;
}

.reaction-type.синтез {
    background: rgba(46, 213, 115, 0.2);
    color: #2ed573;
    border: 1px solid #2ed573;
}

.reaction-type.распад {
    background: rgba(255, 165, 2, 0.2);
    color: #ffa502;
    border: 1px solid #ffa502;
}
.reaction-equation {
    font-family: 'Source Code Pro', monospace;
    font-size: 28px;
    text-align: center;
    padding: 25px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 10px;
    margin: 25px 0;
    border: 1px solid rgba(255, 255, 255, 0.1);
}

.reaction-details {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin: 25px 0;
}

.detail {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 15px;
    background: rgba(255, 255, 255, 0.03);
    border-radius: 8px;
    flex: 1;
    min-width: 250px;
}

.detail i {
    color: #6c5ce7;
    font-size: 20px;
}

.detail-label {
    font-weight: 500;
    color: #aaa;
}

.detail-value {
    font-weight: 500;
}

.reaction-example {
    margin-top: 30px;
    padding-top: 25px;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.chain-reaction {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    gap: 15px;
    margin-top: 20px;
    padding: 20px;
    background: rgba(255, 71, 87, 0.05);
    border-radius: 10px;
}

.nucleus, .fragment, .neutrons {
    padding: 15px 25px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 8px;
    font-family: 'Source Code Pro', monospace;
    font-weight: bold;
}

.arrow {
    font-size: 20px;
    color: #6c5ce7;
}

.fission {
    display: flex;
    align-items: center;
    gap: 10px;
    flex-wrap: wrap;
}

.plus {
    font-size: 20px;
    color: #aaa;
}

.binding-energy {
    background: rgba(255, 255, 255, 0.05);
    padding: 30px;
    border-radius: 15px;
}

.energy-curve {
    position: relative;
    height: 300px;
    margin: 40px 0;
    background: rgba(255, 255, 255, 0.03);
    border-radius: 10px;
}

.curve-point {
    position: absolute;
    transform: translateX(-50%);
}

.point-label {
    width: 40px;
    height: 40px;
    background: #6c5ce7;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    margin-bottom: 10px;
    margin-left: -20px;
}

.point-value {
    position: absolute;
    top: -40px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.7);
    padding: 5px 10px;
    border-radius: 5px;
    font-size: 14px;
    white-space: nowrap;
    opacity: 0;
    transition: opacity 0.3s;
}

.curve-point:hover .point-value {
    opacity: 1;
}

.curve-line {
    position: absolute;
    top: 50px;
    left: 5%;
    width: 90%;
    height: 200px;
}

.curve-line::after {
    content: '';
    position: absolute;
    width: 100%;
    height: 100%;
    border: 2px solid #6c5ce7;
    border-top: none;
    border-radius: 0 0 50% 50%;
}

.curve-explanation {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin: 30px 0;
}

.explanation-item {
    flex: 1;
    min-width: 250px;
    display: flex;
    align-items: center;
    gap: 15px;
    padding: 15px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 10px;
}

.exp-number {
    width: 40px;
    height: 40px;
    background: #6c5ce7;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    font-size: 20px;
    flex-shrink: 0;
}

.energy-applications {
    margin-top: 40px;
}

.energy-applications h3 {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 25px;
    color: #6c5ce7;
}

.applications-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 25px;
}

.application {
    padding: 25px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 10px;
    border-top: 4px solid #2ed573;
}

.application h4 {
    color: #2ed573;
    margin-bottom: 15px;
}

@media (max-width: 768px) {
    .reaction-equation {
        font-size: 20px;
        padding: 15px;
    }
    
    .detail {
        min-width: 100%;
    }
}
</style>
{% endblock %}
{% extends "base.html" %}

{% block title %}Калькулятор - Ядерная физика{% endblock %}

{% block content %}
<div class="container">
    <h1 class="section-title">Калькуляторы ядерной физики</h1>
    
    <div class="calculator-intro">
        <p>Инструменты для расчетов в ядерной физике. Используйте эти калькуляторы для решения задач и изучения формул.</p>
    </div>
    
    <!-- Калькулятор энергии E=mc² -->
    <div class="calculator-section">
        <h2><i class="fas fa-bolt"></i> Калькулятор энергии E=mc²</h2>
        <p>Рассчитайте энергию, эквивалентную массе по знаменитой формуле Эйнштейна.</p>
        
        <div class="calculator-form">
            <div class="input-group">
                <label for="massInput">Масса (кг):</label>
                <input type="number" id="massInput" step="0.001" value="0.001" min="0">
                <div class="input-hint">Пример: 0.001 кг = 1 грамм</div>
            </div>
            
            <button class="btn btn-primary calculate-btn" onclick="calculateEnergy()">
                <i class="fas fa-calculator"></i> Рассчитать энергию
            </button>
            
            <div class="result-container" id="energyResult">
                <!-- Результат появится здесь -->
            </div>
            
            <div class="formula-display">
                <div class="formula">E = mc²</div>
                <p>где c = 299 792 458 м/с (скорость света)</p>
            </div>
            
            <div class="example-calculations">
                <h3>Примеры:</h3>
                <div class="examples">
                    <button class="example-btn" onclick="setExample(0.001, '1 грамм вещества')">1 грамм</button>
                    <button class="example-btn" onclick="setExample(0.000001, '1 миллиграмм вещества')">1 миллиграмм</button>
                    <button class="example-btn" onclick="setExample(1, '1 кг вещества')">1 килограмм</button>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Калькулятор периода полураспада -->
    <div class="calculator-section">
        <h2><i class="fas fa-hourglass-half"></i> Калькулятор радиоактивного распада</h2>
        <p>Рассчитайте оставшееся количество радиоактивного вещества через заданное время.</p>
        
        <div class="calculator-form">
            <div class="input-row">
                <div class="input-group">
                    <label for="initialAmount">Начальное количество (г):</label>
                    <input type="number" id="initialAmount" step="0.01" value="100" min="0">
                </div>
                
                <div class="input-group">
                    <label for="halfLife">Период полураспада (лет):</label>
                    <input type="number" id="halfLife" step="0.01" value="5730" min="0">
                </div>
                
                <div class="input-group">
                    <label for="timePassed">Прошедшее время (лет):</label>
                    <input type="number" id="timePassed" step="0.01" value="1000" min="0">
                </div>
            </div>
            
            <button class="btn btn-primary calculate-btn" onclick="calculateHalfLife()">
                <i class="fas fa-calculator"></i> Рассчитать распад
            </button>
            
            <div class="result-container" id="halfLifeResult">
                <!-- Результат появится здесь -->
            </div>
            
            <div class="formula-display">
                <div class="formula">N(t) = N₀ × (½)<sup>t/T</sup></div>
                <p>где N₀ — начальное количество, t — время, T — период полураспада</p>
            </div>
            
            <div class="example-calculations">
                <h3>Примеры радиоактивных изотопов:</h3>
                <div class="examples">
                    <button class="example-btn" onclick="setHalfLifeExample(100, 5730, 1000, 'Углерод-14: 100г через 1000 лет')">Углерод-14</button>
                    <button class="example-btn" onclick="setHalfLifeExample(100, 1600, 500, 'Радий-226: 100г через 500 лет')">Радий-226</button>
                    <button class="example-btn" onclick="setHalfLifeExample(100, 4.47e9, 1e9, 'Уран-238: 100г через 1 млрд лет')">Уран-238</button>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Калькулятор энергии связи -->
    <div class="calculator-section">
        <h2><i class="fas fa-link"></i> Калькулятор энергии связи ядра</h2>
        <p>Оцените энергию связи атомного ядра по упрощенной формуле Вайцзеккера.</p>
        
        <div class="calculator-form">
            <div class="input-row">
                <div class="input-group">
                    <label for="protonsInput">Число протонов (Z):</label>
                    <input type="number" id="protonsInput" step="1" value="26" min="1" max="120">
                    <div class="input-hint">Атомный номер элемента</div>
                </div>
                
                <div class="input-group">
                    <label for="neutronsInput">Число нейтронов (N):</label>
                    <input type="number" id="neutronsInput" step="1" value="30" min="0" max="180">
                    <div class="input-hint">Массовое число A = Z + N</div>
                </div>
            </div>
            
            <button class="btn btn-primary calculate-btn" onclick="calculateBindingEnergy()">
                <i class="fas fa-calculator"></i> Рассчитать энергию связи
            </button>
            
            <div class="result-container" id="bindingEnergyResult">
                <!-- Результат появится здесь -->
            </div>
            
            <div class="formula-display">
                <div class="formula">E<sub>св</sub> = a<sub>V</sub>A - a<sub>S</sub>A<sup>2/3</sup> - a<sub>C</sub>Z(Z-1)/A<sup>1/3</sup> - a<sub>A</sub>(A-2Z)²/A + δ</div>
                <p>Полуэмпирическая формула Вайцзеккера для энергии связи ядра</p>
            </div>
            
            <div class="example-calculations">
                <h3>Примеры ядер:</h3>
                <div class="examples">
                    <button class="example-btn" onclick="setBindingExample(1, 0, 'Дейтерий (²H)')">Дейтерий</button>
                    <button class="example-btn" onclick="setBindingExample(2, 2, 'Гелий-4 (α-частица)')">Гелий-4</button>
                    <button class="example-btn" onclick="setBindingExample(26, 30, 'Железо-56')">Железо-56</button>
                    <button class="example-btn" onclick="setBindingExample(92, 146, 'Уран-238')">Уран-238</button>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Справочная информация -->
    <div class="reference-section">
        <h2><i class="fas fa-book"></i> Справочные данные</h2>
        
        <div class="reference-grid">
            <div class="reference-card">
                <h3><i class="fas fa-tachometer-alt"></i> Постоянные</h3>
                <ul class="constants-list">
                    <li><strong>c</strong> = 299 792 458 м/с (скорость света)</li>
                    <li><strong>e</strong> = 1.602 × 10⁻¹⁹ Кл (заряд электрона)</li>
                    <li><strong>m<sub>p</sub></strong> = 1.673 × 10⁻²⁷ кг (масса протона)</li>
                    <li><strong>m<sub>n</sub></strong> = 1.675 × 10⁻²⁷ кг (масса нейтрона)</li>
                    <li><strong>1 эВ</strong> = 1.602 × 10⁻¹⁹ Дж (электронвольт)</li>
                    <li><strong>1 МэВ</strong> = 10⁶ эВ</li>
                </ul>
            </div>
            <div class="reference-card">
                <h3><i class="fas fa-calculator"></i> Формулы</h3>
                <ul class="formulas-list">
                    <li><strong>E = mc²</strong> — Эквивалентность массы и энергии</li>
                    <li><strong>N = N₀e<sup>-λt</sup></strong> — Закон радиоактивного распада</li>
                    <li><strong>T<sub>1/2</sub> = ln2/λ</strong> — Период полураспада</li>
                    <li><strong>E<sub>св</sub> = Δmc²</strong> — Энергия связи ядра</li>
                    <li><strong>Q = (m<sub>исх</sub> - m<sub>кон</sub>)c²</strong> — Энергетический выход реакции</li>
                </ul>
            </div>
            
            <div class="reference-card">
                <h3><i class="fas fa-exchange-alt"></i> Единицы измерения</h3>
                <ul class="units-list">
                    <li><strong>Беккерель (Бк)</strong> — активность (1 распад/с)</li>
                    <li><strong>Кюри (Ки)</strong> = 3.7 × 10¹⁰ Бк</li>
                    <li><strong>Грей (Гр)</strong> — поглощенная доза (1 Дж/кг)</li>
                    <li><strong>Зиверт (Зв)</strong> — эквивалентная доза</li>
                    <li><strong>Электронвольт (эВ)</strong> — единица энергии</li>
                </ul>
            </div>
        </div>
    </div>
</div>

<script>
// Установка примера для E=mc²
function setExample(mass, description) {
    document.getElementById('massInput').value = mass;
    calculateEnergy();
}

// Установка примера для периода полураспада
function setHalfLifeExample(initial, halfLife, time, description) {
    document.getElementById('initialAmount').value = initial;
    document.getElementById('halfLife').value = halfLife;
    document.getElementById('timePassed').value = time;
    calculateHalfLife();
}

// Установка примера для энергии связи
function setBindingExample(protons, neutrons, description) {
    document.getElementById('protonsInput').value = protons;
    document.getElementById('neutronsInput').value = neutrons;
    calculateBindingEnergy();
}

// Расчет энергии по формуле E=mc²
async function calculateEnergy() {
    const mass = parseFloat(document.getElementById('massInput').value);
    
    if (isNaN(mass) || mass < 0) {
        showError('energyResult', 'Введите корректное значение массы (≥ 0)');
        return;
    }
    
    try {
        const response = await fetch('/api/calculate-energy', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({ mass: mass })
        });
        
        const data = await response.json();
        
        if (data.success) {
            const result = data.result;
            const html = `
                <div class="result-success">
                    <h3><i class="fas fa-check-circle"></i> Результат расчета:</h3>
                    <div class="result-values">
                        <div class="result-value">
                            <span class="value-label">Энергия:</span>
                            <span class="value">${result.formatted}</span>
                        </div>
                        <div class="result-value">
                            <span class="value-label">В мегаджоулях:</span>
                            <span class="value">${result.megajoules.toFixed(2)} МДж</span>
                        </div>
                        <div class="result-value">
                            <span class="value-label">В килотоннах тротила:</span>
                            <span class="value">${result.kilotons_tnt.toFixed(6)} кт</span>
                        </div>
                    </div>
                    <div class="result-description">
                        Масса ${mass} кг эквивалентна энергии ${result.megajoules.toFixed(2)} МДж,
                        что соответствует взрыву ${result.kilotons_tnt.toFixed(6)} килотонн тротила.
                    </div>
                </div>
            ;
            document.getElementById('energyResult').innerHTML = html;
        } else {
            showError('energyResult', data.error || 'Ошибка расчета');
        }
    } catch (error) {
        showError('energyResult', 'Ошибка соединения с сервером');
    }
}

// Расчет радиоактивного распада
async function calculateHalfLife() {
    const initial = parseFloat(document.getElementById('initialAmount').value);
    const halfLife = parseFloat(document.getElementById('halfLife').value);
    const time = parseFloat(document.getElementById('timePassed').value);
    
    if (isNaN(initial) || initial < 0 || isNaN(halfLife) || halfLife <= 0 || isNaN(time) || time < 0) {
        showError('halfLifeResult', 'Введите корректные значения (начальное количество ≥ 0, период полураспада > 0, время ≥ 0)');
        return;
    }
    
    try {
        const response = await fetch('/api/calculate-half-life', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                initial_amount: initial,
                half_life: halfLife,
                time_passed: time
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            const result = data.result;
            const html = 
                <div class="result-success">
                    <h3><i class="fas fa-check-circle"></i> Результат расчета:</h3>
                    <div class="result-values">
                        <div class="result-value">
                            <span class="value-label">Осталось вещества:</span>
                            <span class="value">${result.formatted} г</span>
                        </div>
                        <div class="result-value">
                            <span class="value-label">Распалось:</span>
                            <span class="value">${result.decayed.toFixed(4)} г</span>
                        </div>
                        <div class="result-value">
                            <span class="value-label">Процент распада:</span>
                            <span class="value">${result.decay_percentage.toFixed(2)}%</span>
                        </div>
                    </div>
                    <div class="result-description">
                        Через ${time} лет из ${initial} г останется ${result.remaining.toFixed(4)} г. 
                        Период полураспада ${halfLife} лет.
                    </div>
                    <div class="decay-stages">
                        <div class="stage">
                            <div class="stage-time">0 лет</div>
                            <div class="stage-amount">${initial} г (100%)</div>
                        </div>
                        <div class="stage">
                            <div class="stage-time">${halfLife} лет</div>
                            <div class="stage-amount">${(initial/2).toFixed(2)} г (50%)</div>
                        </div>
                        <div class="stage">
                            <div class="stage-time">${2*halfLife} лет</div>
                            <div class="stage-amount">${(initial/4).toFixed(2)} г (25%)</div>
                        </div>
                        <div class="stage">
                            <div class="stage-time">${3*halfLife} лет</div>
                            <div class="stage-amount">${(initial/8).toFixed(2)} г (12.5%)</div>
                        </div>
                    </div>
                </div>
            `;
            document.getElementById('halfLifeResult').innerHTML = html;
        } else {
            showError('halfLifeResult', data.error || 'Ошибка расчета');
        }
    } catch (error) {
        showError('halfLifeResult', 'Ошибка соединения с сервером');
    }
}
// Расчет энергии связи
async function calculateBindingEnergy() {
    const protons = parseInt(document.getElementById('protonsInput').value);
    const neutrons = parseInt(document.getElementById('neutronsInput').value);
    
    if (isNaN(protons)  protons < 1  isNaN(neutrons) || neutrons < 0) {
        showError('bindingEnergyResult', 'Введите корректные значения (протоны ≥ 1, нейтроны ≥ 0)');
        return;
    }
    
    try {
        const response = await fetch('/api/calculate-binding-energy', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                protons: protons,
                neutrons: neutrons
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            const result = data.result;
            const A = result.mass_number;
            const elementName = getElementName(protons);
            
            const html = 
                <div class="result-success">
                    <h3><i class="fas fa-check-circle"></i> Результат расчета для ${elementName} (A=${A}):</h3>
                    <div class="result-values">
                        <div class="result-value">
                            <span class="value-label">Энергия связи ядра:</span>
                            <span class="value">${result.formatted}</span>
                        </div>
                        <div class="result-value">
                            <span class="value-label">Удельная энергия связи:</span>
                            <span class="value">${result.specific_energy.toFixed(2)} МэВ/нуклон</span>
                        </div>
                        <div class="result-value">
                            <span class="value-label">Состав ядра:</span>
                            <span class="value">${protons} протонов + ${neutrons} нейтронов</span>
                        </div>
                    </div>
                    <div class="result-description">
                        Энергия связи ядра ${elementName}-${A} составляет ${result.binding_energy.toFixed(2)} МэВ. 
                        Удельная энергия связи ${result.specific_energy.toFixed(2)} МэВ/нуклон.
                    </div>
                    <div class="binding-energy-info">
                        <div class="info-item">
                            <div class="info-label">Стабильность ядра:</div>
                            <div class="info-value">
                                ${result.specific_energy > 8.5 ? 'Высокая' : 
                                  result.specific_energy > 7.5 ? 'Средняя' : 'Низкая'}
                            </div>
                        </div>
                        <div class="info-item">
                            <div class="info-label">Дефект массы:</div>
                            <div class="info-value">~${(result.binding_energy / 931.5).toFixed(4)} а.е.м.</div>
                        </div>
                    </div>
                </div>
            ;
            document.getElementById('bindingEnergyResult').innerHTML = html;
        } else {
            showError('bindingEnergyResult', data.error || 'Ошибка расчета');
        }
    } catch (error) {
        showError('bindingEnergyResult', 'Ошибка соединения с сервером');
    }
}

// Вспомогательные функции
function showError(elementId, message) {
    document.getElementById(elementId).innerHTML = 
        <div class="result-error">
            <i class="fas fa-exclamation-circle"></i> ${message}
        </div>
    ;
}

function getElementName(atomicNumber) {
    const elements = {
        1: 'Водород', 2: 'Гелий', 3: 'Литий', 6: 'Углерод', 8: 'Кислород',
        26: 'Железо', 79: 'Золото', 92: 'Уран', 94: 'Плутоний'
    };
    return elements[atomicNumber] || Элемент Z=${atomicNumber};
}
// Инициализация при загрузке
document.addEventListener('DOMContentLoaded', function() {
    // Автоматически рассчитать первый пример
    calculateEnergy();
    calculateHalfLife();
    calculateBindingEnergy();
});
</script>

<style>
.calculator-intro {
    margin-bottom: 40px;
    text-align: center;
    max-width: 800px;
    margin-left: auto;
    margin-right: auto;
}

.calculator-section {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15px;
    padding: 30px;
    margin-bottom: 40px;
    border-left: 5px solid #6c5ce7;
}

.calculator-section h2 {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 15px;
    color: #6c5ce7;
}

.calculator-form {
    margin-top: 25px;
}

.input-group {
    margin-bottom: 20px;
}

.input-group label {
    display: block;
    margin-bottom: 8px;
    font-weight: 500;
}

.input-group input {
    width: 100%;
    max-width: 400px;
    padding: 12px 15px;
    background: rgba(255, 255, 255, 0.1);
    border: 1px solid #6c5ce7;
    border-radius: 8px;
    color: white;
    font-size: 16px;
}

.input-hint {
    font-size: 14px;
    color: #aaa;
    margin-top: 5px;
}

.input-row {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}

.input-row .input-group {
    flex: 1;
    min-width: 200px;
}

.calculate-btn {
    margin: 20px 0;
    padding: 15px 30px;
    font-size: 18px;
}

.result-container {
    margin: 25px 0;
    min-height: 50px;
}

.result-success, .result-error {
    padding: 20px;
    border-radius: 10px;
    margin-top: 15px;
}

.result-success {
    background: rgba(46, 213, 115, 0.1);
    border: 1px solid #2ed573;
}

.result-error {
    background: rgba(255, 71, 87, 0.1);
    border: 1px solid #ff4757;
    color: #ff4757;
    display: flex;
    align-items: center;
    gap: 10px;
}

.result-success h3 {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 20px;
    color: #2ed573;
}

.result-values {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin-bottom: 20px;
}

.result-value {
    flex: 1;
    min-width: 200px;
    padding: 15px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 8px;
}

.value-label {
    display: block;
    color: #aaa;
    font-size: 14px;
    margin-bottom: 5px;
}

.value {
    font-size: 20px;
    font-weight: bold;
    color: #6c5ce7;
    font-family: 'Source Code Pro', monospace;
}

.result-description {
    margin: 20px 0;
    padding: 15px;
    background: rgba(255, 255, 255, 0.03);
    border-radius: 8px;
    font-style: italic;
}

.decay-stages {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-top: 20px;
}

.stage {
    flex: 1;
    min-width: 150px;
    padding: 15px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 8px;
    text-align: center;
}

.stage-time {
    font-weight: bold;
    color: #6c5ce7;
    margin-bottom: 5px;
}

.stage-amount {
    font-size: 14px;
    color: #aaa;
}

.binding-energy-info {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    margin-top: 20px;
}

.info-item {
    flex: 1;
    min-width: 200px;
    padding: 15px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 8px;
}

.info-label {
    color: #aaa;
    font-size: 14px;
    margin-bottom: 5px;
}

.info-value {
    font-weight: bold;
    color: #6c5ce7;
}

.formula-display {
    margin: 25px 0;
    padding: 20px;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 10px;
    text-align: center;
}

.formula-display .formula {
    font-family: 'Source Code Pro', monospace;
    font-size: 24px;
    margin-bottom: 10px;
    color: #6c5ce7;
}

.example-calculations {
    margin-top: 30px;
}

.example-calculations h3 {
    margin-bottom: 15px;
    color: #aaa;
}

.examples {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}

.example-btn {
    padding: 10px 20px;
    background: rgba(108, 92, 231, 0.2);
    border: 1px solid #6c5ce7;
    border-radius: 6px;
    color: white;
    cursor: pointer;
    transition: all 0.3s;
}

.example-btn:hover {
    background: rgba(108, 92, 231, 0.4);
}
.reference-section {
    margin-top: 50px;
}

.reference-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 25px;
    margin-top: 30px;
}

.reference-card {
    background: rgba(255, 255, 255, 0.05);
    padding: 25px;
    border-radius: 10px;
    border-top: 4px solid #6c5ce7;
}

.reference-card h3 {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 20px;
    color: #6c5ce7;
}

.constants-list, .formulas-list, .units-list {
    list-style: none;
    padding: 0;
}

.constants-list li, .formulas-list li, .units-list li {
    margin-bottom: 10px;
    padding: 8px;
    background: rgba(255, 255, 255, 0.03);
    border-radius: 5px;
}

@media (max-width: 768px) {
    .input-row {
        flex-direction: column;
    }
    
    .result-value, .info-item, .stage {
        min-width: 100%;
    }
}
</style>
{% endblock %}
