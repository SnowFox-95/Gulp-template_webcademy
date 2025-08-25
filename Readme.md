# Сборка Gulp + Автодеплой на Gh-pages

## 1. Установка зависимостей

Чтобы все заработало надо выполнить команду в терминале среды разработки

```bash
npm install --save-dev gulp gulp-sass sass gulp-file-include gulp-clean gulp-server-livereload gulp-sourcemaps gulp-plumber gulp-notify gulp-group-css-media-queries gulp-autoprefixer gulp-csso gulp-replace gulp-htmlclean gulp-imagemin imagemin-webp gulp-svg-sprite gulp-changed gulp-babel @babel/core @babel/preset-env webpack-stream style-loader css-loader gulp-webp-retina-html gulp-web-images-css
```

---

## 2. Что под "капотом" сборки?

- `gulp` — таск-раннер для автоматизации сборки
- `gulp-sass`, `sass` — сборка SCSS/SASS в CSS
- `gulp-file-include` — вставка HTML-фрагментов (include)
- `gulp-clean` — удаление файлов и папок
- `gulp-server-livereload` — локальный сервер с автообновлением страницы
- `gulp-sourcemaps` — создание исходных карт CSS/JS
- `gulp-plumber` — обработка ошибок и уведомления
- `gulp-notify` — обработка ошибок и уведомления
- `gulp-group-css-media-queries` — группировка CSS-медиа-запросов
- `gulp-autoprefixer` — автопрефиксы для CSS
- `gulp-csso` — минификация CSS
- `gulp-replace` — замена текста в потоках
- `gulp-htmlclean` — минификация HTML
- `gulp-imagemin` оптимизация изображений
- `imagemin-webp` — конвертация изображений в WebP
- `gulp-svg-sprite` — генерация SVG-спрайтов
- `gulp-changed` — отслеживание изменений файлов для ускорения сборки
- `gulp-babel`, `@babel/core`, `@babel/preset-env` — транспиляция JS кода
- `webpack-stream`, `style-loader`, `css-loader` — сборка JS-модулей через Webpack
- `gulp-webp-retina-html`, `gulp-web-images-css` — поддержка WebP в HTML и CSS

---

## 3. Быстрый старт

Рекомендуется запускать задачу сборки через среду разработки (например в VSCode: `Терминал - Запуск задачи`).

Но можно делать все и в терминале напрямую (в качестве альтернативы)

### Пример команд для запуска

- `gulp` - запускает сборку в режиме разработки `dev`
- `gulp docs` - запуск сборки в режим публикации (продакшн) с копированием в папку `./docs`

---

## 4. Работа с SCSS: новые правила подключения (***Изменения Август 2025***)

### Структура SCSS

- Все базовые переменные, миксины и утилиты экспортируются через файл `src/scss/base/_index.scss` с помощью `@forward`.
- Все блоки экспортируются через файл `src/scss/blocks/_index.scss` с помощью `@forward`.

### Подключение в main.scss

Вместо устаревших `@import` используйте:

```scss
@use 'base/index' as base;
@use 'blocks/index' as blocks;
```

### Использование переменных и миксинов

- Для обращения к переменным и миксинам используйте namespace:
    - Переменные: `base.$имя_переменной`
    - Миксины: `@include base.имя_миксина`
- Пример:
    ```scss
    @use 'base/index' as base;

    .example {
      color: base.$primary-color;
      @include base.flex-center;
    }
    ```

### Внутри блоков

- Если в блоке используются базовые миксины или переменные, подключайте их через:
    ```scss
    @use '../base/index' as base;
    ```

### Медиа-запросы

- Для размеров используйте SCSS-переменные из `base/vars`:
    ```scss
    @use '../base/vars' as vars;

    @media (max-width: vars.$laptop-size) { ... }
    ```

### Важно

- Не используйте `@import` — только `@use` и `@forward`.
- Все обращения к SCSS-сущностям — только через namespace.

---

**Пример полного подключения:**

```scss
@use 'base/index' as base;
@use 'blocks/index' as blocks;
```

**Пример внутри блока:**

```scss
@use '../base/index' as base;

.block {
  @include base.mobile {
    // ...
  }
}
```

---

**Если используете только CSS custom properties (var(--...)), подключать SCSS-файлы не требуется.**

---

## 5. Автоматический деплой на Github Pages

При пуше в ветку `main`:

- Выполняется сборка в режиме публикации/production (в папку `./docs`)
- Содержимое `./docs` копируется в ветку `gh-pages`
- Github Pages автоматически публикует сайт

---

## 6. Настройка Github actions

В репозитории должен быть workflow-файл `.github/workflows/deploy.yml` c таким содержанием

```yml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout main branch
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npx gulp docs

      - name: Deploy to gh-pages branch
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
          publish_branch: gh-pages
          force_orphan: true

```
---
## 7. Важные моменты
1. Для успешного деплоя нужно убедиться, что в настройках репозитория **включены**:
    - разрешения `contents: write` для GitHub Actions (проверка через *Settings* -> *Actions* -> *General* -> *Workflow permissions*)
2. После первого успешного деплоя необходимо в разделе Settings -> Pages указать ветку `gh-pages` и папку `/(root)`для публикации сайта
---
## 8. Рекомендации и советы

- Все пути в gulp настроены так, что исходники лежат в `./src`, дев-сборка — в `./build`, продакшен сборка — в `./docs`

- `gulp dev` (дефолтная задача) запускает сборку в `./build` с сервером и watcher-ом для разработки

- `gulp docs` запускает production сборку в `./docs` с минификацией, оптимизацией, WebP и т.п.

- Сервер с `livereload` запускается только в дев режиме, в продакшен не используется (во избежание зависания Workflow)

- Для минимизации CSS используется `gulp-csso`, при этом формируются и .css и .min.css файлы

- Для конвертации изображений в WebP используется `imagemin-webp`, обратите внимание на корректность настроек и порядка задач

- Ошибки при деплое чаще всего связаны с правами доступа: проверьте, что в настройках *Actions* репозитория у вас выставлены разрешения **"Read and write permissions"**

- Запуск сборок лучше делать через задачи в редакторе, это удобнее и снижает ошибки при копировании команд
---
## 9. Лицензия 

**MIT License**

---

## 10. Контакты  и помощь

Если возникли вопросы, пожалуйста, обращайтесь к документации Gulp, GitHub Actions или напишите в issue репозитория.
