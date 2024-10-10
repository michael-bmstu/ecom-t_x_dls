# ecom.tech and DLS NLP competition (top 23)

Решение задачи множественной классификации текстов для автоматизации исследования обратной связи пользователей Самоката.

**Структура проекта**

* В папке [data](data) находятся тренировочные и тестовые данные;
* В папке [notebooks](notebooks) находятся 2 варианта решения задачи: [с помощью CatBoost](notebooks/solution_cb.ipynb) и [с помощью Bert](notebooks/solution-bert.ipynb).
* В папке [weights](weights) расположены веса для соответствующих решений [CatBoost](weigths/vcb_weights.npy) и [Bert](weights/bert_weights.pt)

**Результаты соревнования**
- скор CatBoost public: 0.3772
- скор Bert public: 0.4931
- скор Bert private: 0.4954 (top 23)

> Использовалась метрика accuracy по полному совпадению меток классов.

## Описание решения
### Исходные данные
В качестве тренировочных данных предлагались отзывы пользователей сервиса.
- 'assesment' - оценка пользователя (0-6)
- 'tags' - теги выбранные пользователем ('PROMOTIONS','DELIVERY', 'PRODUCTS_QUALITY', 'SUPPORT', 'PAYMENT','PRICE', 'CATALOG_NAVIGATION', 'ASSORTMENT') 
- 'text' - текст отзыва

Предсказывать было нужно принадлежность отзыва к одному из 50 трендов:
- trand_id_res{i} - принадлежность отзва к i-тому тренду (i=0...49)

### Предобработка
Пропущенные значения были заполнены пустой строкой.
Текстовые данные были очищены от мусорных символов, пунктуации и приведены к нижнему регистру.

### CatBoost
Были добавлены новые признаки, отвечающие за количество тегов и длину текста. 

Для решения задачи с помощью CatBoost был обучен ```MultiOutputClassifier``` из библиотеки ```scikit-learn```.
В изначальном решении колонки 'tags' и 'text' были переданы в CatBoost как text_features.

Затем данные о тегах были преобразованы с помощью one-hot-encoding, но это не дало прироста по сравнинию с первоначальным решением.

После обучения веса модели были сохранены. При запуске ноутбука их можно загрузить не выполняя обучение.

### Bert
Для решения задачи была выбрана модель Bert ```BertForSequenceClassification``` 'DeepPavlov/rubert-base-cased'.
В модели был перезадан линейный слой и разморожены веса на последних 8 слоях из 12.

Данные о тексте отзыва, тегах и оценках были сконкатенированны в одну строку и подавались на вход модели.
Теги были заменены на их русскоязычные аналоги, это улучшило качество модели.

#### Обучение модели
1 эпоху обучался только перезаданный линейный слой, остальные слои модели были заморожены, затем 30 эпох обучалась модель с 8 размороженными слоями. По результатам обучения видно, что качество перестало улучшаться после 14 эпохи.

Для обучения линейного слоя:

```criterion = nn.BCEWithLogitsLoss()```

```optimizer_l = torch.optim.Adam()```

Для обучения полной модели:

```criterion = nn.BCEWithLogitsLoss()```

```optimizer = torch.optim.Adam()```

```scheduler = get_cosine_schedule_with_warmup()``` из библиотеки ```transformers```

