# kadastralna_karta — карта за избор на имоти (Telegram Mini App)

Статична MapLibre карта на кадастрални имоти от отворените данни на АГКК. Отваря се като
**Telegram Web App** от бутон в бота `protokol_app`: геодезистът тапва имоти на картата,
натиска „Генерирай" и изборът се връща към бота (`Telegram.WebApp.sendData`), който генерира
трасировъчните данни.

Целият frontend е един файл (`index.html`) + данни в `data/` + шрифтове в `fonts/`.
Без build стъпка — каквото е в repo-то, това се сервира.

## Структура

```
index.html                         целият UI + логика (MapLibre 5.7.3 от cdnjs)
.nojekyll                          ИЗКЛЮЧВА Jekyll — задължителен (виж по-долу)
data/
  <ekatte>.geojson                 имоти на землище (контури + cadnum + атрибути)
  <ekatte>_raioni.geojson          контури + номера на кадастрални райони
  <ekatte>_sgradi.geojson          сгради (за ориентация)
  granici.geojson                  национален слой землищни граници (~6 MB)
fonts/Open Sans Regular/*.pbf      глифове за етикетите (латиница/цифри + кирилица)
```

Данните се генерират от `kais-otvoreni-danni/scripts/geojson_export.py` в основния проект.

## Деплой на GitHub Pages

Repo: **`mdemerdzhiev/kadastralna_karta`**. Съдържанието на тази папка (`web_map/`) е
**root** на repo-то.

```bash
# в папката web_map/
git init -b main
git add -A                 # увери се, че .nojekyll и fonts/ влизат
git commit -m "kadastralna karta — initial"
gh repo create mdemerdzhiev/kadastralna_karta --private --source=. --remote=origin --push
```

После в GitHub: **Settings → Pages → Build and deployment → Deploy from a branch →
`main` / `/ (root)`**. След минута сайтът е на:

```
https://mdemerdzhiev.github.io/kadastralna_karta/
```

Provери, че URL-ът **със завършваща наклонена черта** зарежда картата с етикети (имоти,
райони, граници). Когато всичко работи — направи repo-то публично (Pages на безплатен план
изисква публичен repo, освен ако нямаш GitHub Pro).

## ⚠️ Две неща, които ЧУПЯТ деплоя, ако се пропуснат

1. **`.nojekyll` е задължителен.** Без него GitHub пуска Jekyll, който може да не сервира
   папката `fonts/Open Sans Regular/` (има интервал в името) → етикетите изчезват. Файлът е
   празен; важи само наличието му.

2. **Trailing slash в URL-а на бота.** Всички пътища в `index.html` са относителни
   (`data/...`, `fonts/...`), затова `TELEGRAM_MINI_APP_URL` в `config.py` ТРЯБВА да завършва
   с `/`:

   ```
   ✅ https://mdemerdzhiev.github.io/kadastralna_karta/
   ❌ https://mdemerdzhiev.github.io/kadastralna_karta
   ```

   Без наклонената черта относителните пътища резолват към грешна база и данните дават 404.

## Кеширане / ъпдейт след деплой

GitHub Pages (Fastly CDN) и Telegram WebView кешират агресивно — след `git push` нов
`index.html` може да се вижда чак след време. За незабавен cache-bust добави версия към URL-а
в `config.py`:

```
https://mdemerdzhiev.github.io/kadastralna_karta/?v=2
```

(увеличавай числото при всеки деплой, който трябва да стигне до телефона веднага).

## Дебъг на терен

Диагностиката (зелен надпис долу вляво + версия в заглавието) е **скрита по подразбиране**.
Включва се с `?debug` в URL-а:

```
https://mdemerdzhiev.github.io/kadastralna_karta/?debug
```

Производственият URL (без `?debug`) показва чисто UI.

## Локален тест

```bash
python -m http.server 8000 --directory web_map
# отвори http://localhost:8000/?debug
```

НЕ отваряй `index.html` директно като файл (`file://`) — MapLibre Web Worker не тръгва оттам
и относителните `fetch` гърмят.

## Добавяне на ново землище

Сложи `<ekatte>.geojson` (+ по желание `_raioni` / `_sgradi`) в `data/`, commit, push.
Картата ги зарежда при тап върху землището в националния граничен слой.
