# Домашнее задание №4

В этом задании требуется обкачать интернет-магазин компьютерных игр ["GG.DEALS"](https://gg.deals/) с использованием библиотек [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) и/или [lxml](https://lxml.de/). Ваша программа должна скачать информацию о самых [рейтинговых](https://gg.deals/games/?sort=metascore&type=1) 300 играх магазина. 

## Общий подход к решению задачи

Задачу условно можно разделить на два этапа. На первом этапе требуется получить ссылки на все нужные игры из раздела, на втором – получить информацию о каждой из игр.

### Этап 1. Получение ссылок на игры

Для получения всех ссылок на игры нужно обкачать первые страницы [раздела](https://gg.deals/games/?sort=metascore&type=1). Переключаясь между страницами раздела можно заметить, как меняется URL страницы (появляется параметр `page`). Варьируя это значение в диапазоне можно получить ссылки на все требуемые страницы.

### Этап 2. Получение информации об игре

Рассмотрим в качестве примера [карточку](https://gg.deals/game/tricky-towers/) игры «Tricky Towers».

![скриншот](screenshot_long.png)

На скриншотах выделены 7 областей, из каждой области требуется извлечь следующие элементы:

0. поле "url" – url страницы из адресной строки браузера;
1. поле "name" – название игры;
2. поле "image" – ссылка на постер игры; поле "market_url" – ссылка на игру в оригинальном магазине (см. View On Steam, например);
3. поля "wishlist_count", "alert_count", "owners_count" – значения соответвующих счетчиков.
4. группы полей, если имеются:
	* "release_date" – дата релиза (выхода) игры;
	* "developer" – разработчик игры;
	* "metacritic_score" – рейтинг Metascore;
	* "user_score" – рейтинг Userscore;
	* "review_label", "review_positive_pctg", "review_count" – общий пользовательский вердикт (например, Very Positive), доля позитивных обзоров, общее число обзоров на игру;
	* "genres" – список жанров игры;
	* "tags" – список тегов игры;
	* "features" – список особенностей игры;
    * "platforms" – список поддерживаемых ОС.
5. поле "dlcs" – список ссылок на DLC (дополнения) к игре, поле "packs" – список ссылок на Packs (расширенные версии игр); списки могут быть пустыми;
6. поле "price_history" – список цен на игру за весь имеющийся период. 
 
Список цен на игру должен представлять собой питоновский список словаре	й, каждый словарь должен иметь три поля:
* "ts" – время изменения цены;
* "price" – новая цена (в долларах);
* "shop" – имя магазина.

Таким образом карточка из примера представима в виде следующего json-а:
```json
{
    "url": "https://gg.deals/game/tricky-towers/",
    "status": true,
    "name": "Tricky Towers",
    "image": "https://img.gg.deals/00/ee/c8582c4fa19ba60498d50c1b6cfbc859afaa_307xt176.jpg",
    "release_date": "02 Aug 2016",
    "developer": "WeirdBeard",
    "metacritic_score": 80,
    "user_score": 6.9,
    "review_label": "Very Positive",
    "review_positive_pctg": 90,
    "review_count": 8736,
    "genres": [
        "Casual",
        "Indie"
    ],
    "tags": [
        "Party Game",
        "Puzzle",
        "Physics",
        "Casual",
        "Building",
        "Multiplayer",
        "Funny",
        "Family Friendly",
        "Cartoony",
        "Arcade",
        "Local Multiplayer",
        "Singleplayer",
        "Online Co-Op",
        "Local Co-Op",
        "PvP",
        "4 Player Local",
        "Strategy",
        "Party",
        "Colorful",
        "Stylized",
        "Indie"
    ],
    "features": [
        "Single-player",
        "Online PvP",
        "Shared/Split Screen PvP",
        "Steam Achievements",
        "Full controller support",
        "Steam Trading Cards",
        "Steam Leaderboards",
        "Remote Play on Phone",
        "Remote Play on Tablet",
        "Remote Play on TV",
        "Remote Play Together"
    ],
    "platforms": [
        "Windows",
        "Mac",
        "Linux",
        "Steam Deck Verified",
        "GeForce Now"
    ],
    "wishlist_count": 857,
    "alert_count": 76,
    "owners_count": 5334,
    "market_url": "https://store.steampowered.com/app/437920/",
    "dlcs": [
        "https://gg.deals/dlc/tricky-towers-indie-friends-pack/",
        "https://gg.deals/dlc/tricky-towers-spirit-animal-pack/",
        "https://gg.deals/dlc/tricky-towers-galaxy-bricks/",
        "https://gg.deals/dlc/tricky-towers-candy-bricks/",
        "https://gg.deals/dlc/tricky-towers-gem-bricks/",
        "https://gg.deals/dlc/tricky-towers-holographic-bricks/",
        "https://gg.deals/dlc/tricky-towers-original-soundtrack/"
    ],
    "packs": [],
    "price_history": [
        {
            "ts": 1470140059,
            "price": 12.74,
            "shop": "Steam"
        },
        {
            "ts": 1470762964,
            "price": 14.99,
            "shop": "Steam"
        },
        {
            "ts": 1473790235,
            "price": 11.99,
            "shop": "Humble Store"
        },
        {
            "ts": 1474395110,
            "price": 14.99,
            "shop": "Humble Store"
        },
        {
            "ts": 1477675888,
            "price": 10.04,
            "shop": "Steam"
        }
    ]
}
```

Часть значений в "price_history" пропущены для наглядности.

**Замечание:** Не нужно создавать поля со значением `null` (python-ий `None`) или пустыми строками. Сохраняйте только те поля, которые есть у карточки.

Далее будем полагать, что у нас есть функция `process_page(url)`, которая для карточки с ссылкой `url` возвращает описанный выше словарь.

Для удобства вам предоставлен файл [`example.jsonl`](example.jsonl) с результатами парсинга некоторых карточек.

#### Параллелизация обкачки

Так как большую часть времени парсинга занимают скачивание карточки (I/O-операция) и обращения к библиотекам парсинга (имеют основу, написанную на C++), можно воспользоваться параллелизацией с использованием потоков.

Для того, чтобы не хранить все скачанные карточки в оперативной памяти, посадим каждый поток на свой файл:
* поток зачитывает URL из очереди;
* скачивает и парсит страницу;
* результат сохраняет в файл.

Данную логику можно реализовать, например, следующим образом:

```python
import gzip
import json
import codecs

from multiprocessing.dummy import Pool, Queue

queue = Queue(...)   # очередь ссылок на книги


def process_page_wrapper(i):
    with gzip.open('data/part_{:05d}.jsonl.gz'.format(i), mode='wb') as f_json:
        f_json = codecs.getwriter('utf8')(f_json)

        while not queue.empty():
            # из process_page может вылететь исключение
            record = process_page(queue.get())
            record_str = json.dumps(record, ensure_ascii=False)
            print(record_str, file=f_json)

            # счетчик должен атомарно обновиться
            with lock:
                pbar.update(1)


with Pool(processes=8) as pool, tqdm(total=queue.qsize()) as pbar:
    lock = pbar.get_lock()
    pool.map(process_page_wrapper, range(pool._processes))
    pool.join()
    pool.close()
```

**Замечание:** приведенная выше реализация не является законченной, требуется добавить логирование ошибок (какая страница не смогла быть обработана и почему). Для логирования можно использовать `print` или модуль `logging`:
```python
print(message, file=sys.stderr)
```
Если Вы используете `print`, то забывайте при логировании использовать `lock`.

## Общие рекомендации по выполнению задания

1. В данном задании вы строите небольшую, но все же, систему для обкачки сайтов. Поэтому полезно мониторить ее состояние, например, писать сколько книг уже было обработано. Для этого можно пользоваться библиотекой [tqdm](https://tqdm.github.io/). 
2. Не смешивайте этапы задания вместе. Сначала получите все url-ы карточек, а затем для каждой карточки извлеките требуемую информацию.
3. Используйте возможности параллельной работы с данными Python, например, модуль multiprocessing.
4. Помните, что парсинг страниц очень нестабильный процесс, поэтому требуется сделать ваш код максимально безопасным: явно проверять извлечение каждого из полей (делать соответствующие проверки), проверять что страница скачана (если страница не скачана, то попробовать ее перекачать) и т.п.
5. Если страницу не получается скачать, попробуйте скачать ее еще раз. Например, можно реализовать функцию:
```python
def get_page(url, n_attempts=5, t_sleep=1, **kwargs):
    pass
```
6. Старайтесь не искать каждый элемент по ВСЕЙ странице. Если элементы расположены в одном блоке, то лучше сначала найти этот общий блок, а затем элементы уже внутри этого блока.

