---
title: Лекции по фронтенду - Redux Router
---

## Redux Router

![redux router](assets/redux-router/router-meme.png)

[Дмитрий Вайнер](https://github.com/dmitryweiner)

---

### Идея
* Нужно сделать одностраничное приложение с несколькими страницами.
* При переходе между страницами должна меняться адресная строка.
* Если не используется Redux, можно спокойно использовать [react-router-dom](https://dmitryweiner.github.io/lectures/React%20Router.html#/).
* Надо подружить роутер и Redux, чтобы можно было менять роуты не только из компонентов, но и из экшенов и других мест.
* Для этого роут и история хранятся в сторе!

---

### Выбор библиотеки
* Раньше бы я порекомендовал [react-router-redux](https://github.com/reactjs/react-router-redux).
* Но её разработка прекращена в 2018 году.
* Автор советует переходить на [connected-react-router](https://github.com/supasate/connected-react-router).
* Её и будем использовать.
* [Эта библиотека не единственная](https://github.com/markerikson/redux-ecosystem-links/blob/master/routing.md).

---

### Подключение: первый этап
* Установка:

```shell
npm i connected-react-router
npm i -D @types/history # опционально, для использующих TS
```
* Создание объекта ```history```:

```js
// для standalone хостинга
import { createBrowserHistory } from 'history';
// для хостинга в каталоге
import { createHashHistory } from 'history';
const history = createBrowserHistory();
```

---

### Подключение: второй этап
* Создание стора:

```js
import { combineReducers, createStore, applyMiddleware } from 'redux';
import { connectRouter, routerMiddleware } from 'connected-react-router';
const store = createStore(
    combineReducers({
        router: connectRouter(history), // ключ объекта только "router", не иначе
        ... // rest of your reducers
    }),
    applyMiddleware(
        routerMiddleware(history), // for dispatching history actions
        // ... other middlewares ...
    )
);
```

---

### Подключение: третий этап
* Оборачиваем ```<App>``` в ```<ConnectedRouter>```:
```js
import { ConnectedRouter } from 'connected-react-router';
ReactDOM.render(
    <Provider store={store}>
        <ConnectedRouter history={history}>
            <App />
        </ConnectedRouter>
    </Provider>,
    document.getElementById('root')
);
```

---

### Switch
* Внутри ```<App>``` используется компонент ```<Switch>```, в котором лежат нужные роуты.
* [Ознакомиться c работой Route, Switch](https://dmitryweiner.github.io/lectures/React%20Router.html#/6).
```js
import { Route, Switch } from 'react-router';
//
<Switch>
    <Route exact path="/" render={() => (<div>Match</div>)} />
    <Route render={() => (<div>Miss</div>)} />
</Switch>
```
* Установка:
```shell
npm i react-router react-router-dom
npm i -D @types/react-router @types/react-router-dom # для TS
```

---

### Переход по роутам
* В компоненте:
```js
import { Link } from 'react-router-dom';
//
<Link to="/home">Home</Link>
```
* Или так:
```js
import { push } from 'connected-react-router';
//
<div onClick={() => {
    push('/login');
}}>login</div>
```  

---

### Переход по роуту из асинхронного экшена
```js
import { push } from 'connected-react-router'
//
export const login = (username, password) => (dispatch) => {
  /* do something before redirection */
  dispatch(push('/home'))
};
```

---

### Переход из саги
```js
import { push } from 'connected-react-router'
import { put, call } from 'redux-saga/effects'
//
export function* login(username, password) {
  /* do something before redirection */
  yield put(push('/home'))
}
```

---

### Узнать текущие параметры роутера
* Они лежат в сторе:
```js
state.router.location.pathname // текущий URL
state.router.location.search // поисковый запрос, разобранный на параметры
state.router.location.hash // то, что после # в URL
```
* Для ```/hello?a=1#123```:
```js
location = {
    pathname: "/hello",
    search: "?a=1",
    hash: "#123",
    query: {
        a: "1"
    }
}
```

---

### Как получить параметры из роута ```route/:id```
* Всё очень плохо, разработчики предлагают костыли. 
  Можно сходить поставить 👍 за нормальное решение в 
  [issue](https://github.com/supasate/connected-react-router/issues/38).
* Костыль получше:

```tsx
import { createMatchSelector } from 'connected-react-router';
import { useSelector } from 'react-redux';
const matchSelector = createMatchSelector<AppState, {id: string}>({ path: '/user/:id' });
function User() {
    const match = useSelector(matchSelector);
    return <>{match?.params.id}</>;
}
```

----

* Костыль похуже:

```tsx
import { matchPath } from 'react-router-dom';
import { useSelector } from 'react-redux';
function User() {
    const pathname = useSelector((state: AppState) => state.router.location.pathname);
    const match = matchPath<{id: string}>(
        pathname, // like: /user/123
        { path: '/user/:id' }
    );
    return <>{match?.params.id}</>;
}
```

---

### Полезные ссылки
* [Документация](https://github.com/supasate/connected-react-router).
* [ЧаВо](https://github.com/supasate/connected-react-router/blob/master/FAQ.md).
* [Примеры](https://github.com/supasate/connected-react-router/blob/master/examples).
* [Список возможных роутеров](https://github.com/markerikson/redux-ecosystem-links/blob/master/routing.md).
