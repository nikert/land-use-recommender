# land-use-recommender

Алгоритм рекомендации видов разрешённого использования (ВРИ) земельного
участка по Классификатору Росреестра № П/0412. На вход подаётся участок,
на выходе — ранжированный список из укрупнённых категорий ВРИ,
отсортированных по пригодности.

Решение опирается на пять характеристик участка двух типов.
**Численные характеристики** — уклон рельефа, насыщенность окружения
точками интереса и удалённость от автодорог — обрабатываются методом,
который восстанавливает плавную кривую распределения каждого признака
внутри каждого ВРИ и оценивает, насколько новое значение для неё
типично. **Категориальные характеристики** — категория земель и попадание
в зоны с особыми условиями использования территорий (ЗОУИТ — санитарные,
охранные, водоохранные и т. п.) — обрабатываются через частоту
встречаемости каждого варианта внутри каждого ВРИ. Полученные оценки
объединяются в общий рейтинг похожести нового участка на типичных
представителей каждого класса.

Модель обучена на 196 695 участках Республики Алтай, апробирована
на Манжерокском сельском поселении Майминского района.

Полное техническое описание метода — в [`docs/method.md`](docs/method.md).

---

## Состав репозитория

```
land-use-recommender/
├── README.md
├── LICENSE
├── CITATION.cff           ← академическая цитата
├── requirements.txt
├── .gitignore
├── pipeline.py            ← основной конвейер
├── data/
│   ├── README.md
│   ├── classifier_vri.csv ← Классификатор № П/0412 (171 позиция)
│   └── boundary_1/2/3.geojson ← пилотные контуры (Манжерок)
├── results/
│   ├── README.md
│   └── results.json       ← эталонный вывод конвейера
└── docs/
    └── method.md          ← расширенное описание метода
```

Тяжёлые геопространственные слои (`egrn`, `zouit`, `roads`, `poi`, `dem`)
опубликованы в [GitHub Release `data-v1`](https://github.com/nikert/land-use-recommender/releases/tag/data-v1) — см. ниже.

---

## Установка

```bash
git clone https://github.com/nikert/land-use-recommender.git
cd land-use-recommender
python -m venv .venv
source .venv/bin/activate              # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Требуется Python ≥ 3.10. Зависимости — `geopandas`, `shapely`, `scipy`,
`rasterio`, `pyproj`, `matplotlib`, `numpy` (полный список —
в `requirements.txt`).

На Linux и macOS установка `geopandas` и `rasterio` через `pip` обычно
работает из коробки. На Windows может потребоваться conda:

```bash
conda install -c conda-forge geopandas rasterio
```

---

## Подготовка данных

Малые файлы (классификатор и пилотные контуры) уже лежат в `data/`.
Тяжёлые слои нужно догрузить из релиза [`data-v1`](https://github.com/nikert/land-use-recommender/releases/tag/data-v1):

```bash
cd data
REL=https://github.com/nikert/land-use-recommender/releases/download/data-v1
curl -LO $REL/dem.tif
curl -LO $REL/egrn.geojson
curl -LO $REL/poi_point.geojson
curl -LO $REL/poi_polygon.geojson
curl -LO $REL/roads.geojson
curl -LO $REL/zouit.geojson
cd ..
```

После этого в `data/` будут все слои, ожидаемые конвейером.

| Файл | Размер | Источник |
| :--- | ---: | :--- |
| `classifier_vri.csv` | 100 КБ | Приказ Росреестра № П/0412 |
| `boundary_1/2/3.geojson` | < 5 КБ | Пилотные контуры в Манжероке |
| `dem.tif` | 138 МБ | SRTM 1 arc-second / Copernicus GLO-30 |
| `egrn.geojson` | 564 МБ | Открытые фрагменты ЕГРН по Республике Алтай |
| `zouit.geojson` | 69 МБ | Открытые данные Росреестра, агрегированы в 10 типов |
| `roads.geojson` | 31 МБ | OpenStreetMap |
| `poi_point.geojson` | 2 МБ | OpenStreetMap |
| `poi_polygon.geojson` | 3 МБ | OpenStreetMap |

Контрольные суммы SHA-256 указаны на странице релиза.

---

## Запуск

```bash
python pipeline.py data/
```

Конвейер создаёт папку `data/results_YYYY-MM-DD_HH-MM-SS/`, в которую
сохраняет апостериоры, P̃, Ẽ, S и метрики CV в `results.json`,
12 диагностических SVG-диаграмм и 7 PNG-картограмм пилотных контуров.

Эталонный `results.json` от 13 апреля 2026 г. лежит в `results/`.

---

## Цитирование

Если работа использовалась в исследовании или публикации:

```bibtex
@mastersthesis{dmitriev2026vri,
  author    = {Дмитриев, Никита Вячеславович},
  title     = {Разработка метода выбора разрешённого использования
               территории проектирования},
  school    = {Университет ИТМО, Институт дизайна и урбанистики},
  year      = {2026},
  address   = {Санкт-Петербург},
  url       = {https://github.com/nikert/land-use-recommender}
}
```

GitHub также отображает кнопку **Cite this repository** в правой
колонке — она читает `CITATION.cff` и предлагает APA, BibTeX
и другие форматы.

---

## Лицензия

Код распространяется под лицензией [MIT](LICENSE) — свободное
использование, изменение и распространение при сохранении уведомления
об авторстве.

Геопространственные данные в релизе `data-v1` публикуются в
исследовательских целях со ссылкой на первоисточники (Росреестр,
OpenStreetMap, открытые DEM-данные).

---
