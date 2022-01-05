---
title: 像mobx一样，用react tree 外部的变量更新react组件
tags: [react]
date: 2022-01-05
---

看到一个 mobx 的例子很有意思

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { makeAutoObservable } from 'mobx';
import { observer } from 'mobx-react';

// Model the application state.
class Timer {
    secondsPassed = 0;

    constructor() {
        makeAutoObservable(this);
    }

    increase() {
        this.secondsPassed += 1;
    }

    reset() {
        this.secondsPassed = 0;
    }
}

const myTimer = new Timer();

// Build a "user interface" that uses the observable state.
const TimerView = observer(({ timer }) => (
    <button onClick={() => timer.reset()}>Seconds passed: {timer.secondsPassed}</button>
));

ReactDOM.render(<TimerView timer={myTimer} />, document.body);

// Update the 'Seconds passed: X' text every second.
setInterval(() => {
    myTimer.increase();
}, 1000);
```

通常一个 react 状态库，根状态需要注册在 react tree 内部，mobx 很有意思，状态可以在 tree 外部。

自己实现看看如何用外部变量更新 react

```js
import React, { useState } from 'react';
import ReactDOM from 'react-dom';
import './index.css';

// Model the application state.
class Timer {
    secondsPassed = 1;

    increase() {
        this.secondsPassed += 1;
    }

    reset() {
        this.secondsPassed = 0;
    }
}

const myTimer = new Timer();

function observer(component) {
    return ({ timer }) => {
        const [, setState] = useState();
        const forceUpdate = () => setState([]);

        if (!timer.__increase) {
            timer.__increase = true;
            const originFunc = timer.increase.bind(timer);
            timer.increase = () => {
                originFunc();
                forceUpdate();
            };
        }

        return component({ timer });
    };
}

const TimerView = observer(({ timer }) => (
    <button onClick={() => timer.reset()}>Seconds passed: {timer.secondsPassed}</button>
));

ReactDOM.render(<TimerView timer={myTimer} />, document.getElementById('root'));

setInterval(() => {
    myTimer.increase();
}, 1000);
```

一个超简单，耦合度超高的定制版本，后期看看 mobx 如何做的，先占个坑
