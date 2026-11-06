# svojak

## Вопросы игры

Вопросы загружаются из `public/questions.json`. Файл содержит массив `categories`, у каждой категории есть `name` и массив `questions`. Вопрос задаётся полями:

```json
{
  "value": 100,
  "text": "Текст вопроса или путь к изображению",

  "answer": "Правильный ответ или путь к изображению",
  "answerImage": false
}
```

Для изображения укажите `isImage: true` или `answerImage: true`, а в соответствующем поле — путь от корня сайта, например `/images/earth.svg`. Изображения разместите в `public/images/`. Для текста альтернативного описания изображения вопроса можно добавить поле `imageAlt`.

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Lints and fixes files

```
npm run lint
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).
