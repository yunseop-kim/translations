# 바닐라 자바스크립트로 상태 관리 시스템 구축

BY [ANDY BELL ](https://css-tricks.com/author/andybell/)ON JULY 25, 2018

상태 관리는 소프트웨어에서 새로운 것이 아니지만, 자바스크립트로 소프트웨어를 구축할 때는 상대적으로 생소하게 느겨질 것입니다. 전통적으로 우리는 DOM 자체에서 상태를 관리했거나, 윈도우 전역 객체에 할당하기도 했습니다. 그러나 이제는 라이브러리와 프레임워크로 이 문제를 해결 할 수 있습니다. Redux, MobX, Vuex 같은 라이브러리들은 컴포넌트 간 상태 관리를 손쉽게 관리할 수 있습니다. 이는 어플리케이션의 탄력성(resilience)에 적합하며 React 또는 Vue와 같은 상태 기반의 반응형 프레임워크에서 잘 동작합니다.

이 라이브러리들은 어떻게 작동할까요? 우리가 스스로 코드를 작성하려면 무엇이 필요할까요? 파헤쳐보면 간단하게 몇 가지 일반적인 패턴을 배우고 우리에게 유용한 API에 대해서 배울 수 있습니다.

시작하기 전에, 당신이 자바스크립트에 대해 중급자 이상의 지식이 있는 것이 좋습니다. 데이터 타입에 대해 알아야 하며, ES6 이상의 자바스크립트 기능에 대한 지식이 있어야 합니다. 그렇지 않다면 [이곳](https://css-tricks.com/learning-gutenberg-4-modern-javascript-syntax/)에서 선행 학습을 해주세요. Redux 나 MobX를 이걸로 대체해야한다는 말은 아닙니다. 우리는 스킬 업을위한 약간의 프로젝트를 진행할 것입니다. JavaScript 페이로드의 크기를 신경쓰고 있다면 작은 애플리케이션에 확실히 도움이 될 수 있습니다.

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-0)시작하기

코드 속으로 빠져들기 전에, 우리가 무엇을 만드는지 확인해 봅시다. "한 일 리스트"는 여러분이 오늘 달성한 것들을 추가시키는 앱입니다. 프레임워크의 의존성 없이 다양한 UI요소를 마술같이 업데이트 할 겁니다. 사실 마술은 아닙니다. Behind the scenes, we’ve got a little state system that’s sitting, waiting for instructions and maintaining a single source of truth in a predictable fashion.

[데모](https://vanilla-js-state-management.hankchizljaw.io/)

[저장소](http://github.com/hankchizljaw/vanilla-js-state-management)

멋지죠? 관리를 먼저 해 봅시다. 해당 튜토리얼을 잘 유지할 수 있도록 약간의 보일러플레이트를 넣었습니다. 여러분이 해야 할 첫번째 일은 [GitHub에서 클론](https://github.com/hankchizljaw/vanilla-js-state-management-boilerplate) 하거나 [ZIP 아카이브](https://github.com/hankchizljaw/vanilla-js-state-management-boilerplate/archive/master.zip)를 다운로드 한 뒤 압축을 푸는 것입니다.

작업을 마치면 이제 로컬 웹 서버를 실행해야 합니다. 저는 [http-server](https://www.npmjs.com/package/http-server)와 같은 패키지를 이용할 것이지만, 여러분의 입맛에 맞게 서버를 사용해도 됩니다. 로컬로 실행하면 다음과 같은 내용이 표시됩니다:

![img](https://cdn.css-tricks.com/wp-content/uploads/2018/07/state-js-1.png)보일러 플레이트의 초기 상태입니다.

#### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-1)구조 설정

당신이 선호하는 텍스트 에디터의 루트 디렉토리를 여세요. 제 루트 폴더는 다음과 같습니다.

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/
```

다음과 같은 구조를 확인하세요.

```
/src
├── .eslintrc
├── .gitignore
├── LICENSE
└── README.md
```

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-2)Pub/Sub

다음으로, `src` 폴더를 연 다음 거기에 있는 js 폴더를 여세요. `lib` 이라는 새 폴더를 만드세요. 그 안에 `pubsub.js` 라는 파일을 만드세요.

`js ` 폴더의 구조는 아래와 같습니다.

```
/js
├── lib
└── pubsub.js
```

Open up `pubsub.js` because we’re going to make a little [Pub/Sub pattern](https://msdn.microsoft.com/en-us/magazine/hh201955.aspx), which is short for “Publish/Subscribe." We’re creating the functionality that allows other parts of our application to subscribe to named events. Another part of the application can then publish those events, often with some sort of relevant payload.

작은 [Pub/Sub 패턴](https://msdn.microsoft.com/en-us/magazine/hh201955.aspx) ("Publish/Subscribe"의 줄임말) 을 만들 것이므로 pubsub.js를 여십시오. 우리는 어플리케이션의 어느 한 부분을 명명 된 이벤트로 구독 할 수 있는 기능을 만들고 있습니다. 그런 다음 어플리케이션의 이벤트는 종종 관련 페이로드와 함께 관련된 이벤트를 게시 할 수 있습니다.

Pub / Sub는 때로는 이해하기가 쉽지 않으므로 비유는 어떨까요? 당신은 레스토랑에서 일하고 있으며, 고객이 에피타이저와 메인 코스를 갖고 있다고 상상해보세요. 레스토랑에서 일한 적이 있다면, 서빙하는 사람이 에피타이저를 치울 때, 요리사가 어느 테이블의 에피타이저를 비우는지 알게 됩니다. 이것은 해당 테이블의 메인 코스 요리의 시작을 알리는 단서입니다. 큰 레스토랑에서는 요리사가 몇 명이 있어서 다른 요리를 먹을수도 있습니다. 요리사들은 모두 고객이 에피타이저를 다 먹었다는 단서를 서빙하는 사람에게서 *구독*하므로 메인 코스 요리를 준비하는 *기능*을 수행합니다. 따라서 서로 다른 기능 (콜백)을 수행하기 위해 동일한 단서 (이벤트)에서 기다리는 여러 명의 요리사가 있습니다.

![img](https://cdn.css-tricks.com/wp-content/uploads/2018/07/state-management-restaurant.jpg)

Hopefully thinking of it like that helps it make sense. Let’s move on!

이렇게 생각하면 도움이 될 것이라 생각합니다. 계속 가시죠!

The PubSub pattern loops through all of the subscriptions and fires their callbacks with that payload. It’s a great way of creating a pretty elegant reactive flow for your app and we can do it with only a few lines of code.

Pub/Sub 패턴은 모든 구독을 돌며 해당 페이로드로 콜백을 발생시킵니다. 이는 앱에 대한 매우 우아한 반응형 흐름을 만드는 가장 좋은 방법이며 몇 줄의 코드만으로도 해결할 수 있습니다.

 `pubsub.js`에 다음을 추가합니다.

```
export default class PubSub {
  constructor() {
    this.events = {};
  }
}
```

What we’ve got there is a fresh new class and we’re setting `this.events` as a blank object by default. The `this.events` object will hold our named events.

After the constructor's closing bracket, add the following:

 우리가 가지고있는 것은 새롭고 새로운 클래스이며 우리는 기본적으로 `this.events`를 빈 객체로 설정합니다. `this.events` 객체는 명명 된 이벤트를 보유합니다. 생성자의 닫는 괄호 뒤에 다음을 추가합니다

constructor의 닫는 괄호 뒤에 다음을 추가합니다.

```
subscribe(event, callback) {

  let self = this;

  if(!self.events.hasOwnProperty(event)) {
    self.events[event] = [];
  }

  return self.events[event].push(callback);
}
```

This is our subscribe method. You pass a string `event`, which is the event’s unique name and a callback function. If there’s not already a matching event in our `events` collection, we create it with a blank array so we don’t have to type check it later. Then, we push the callback into that collection. If it already existed, this is all the method would do. We return the length of the events collection, because it might be handy for someone to know how many events exist.

Now that we’ve got our subscribe method, guess what comes next? You know it: the `publish` method. Add the following after your subscribe method:

```
publish(event, data = {}) {

  let self = this;

  if(!self.events.hasOwnProperty(event)) {
    return [];
  }

  return self.events[event].map(callback => callback(data));
}
```

This method first checks to see if the passed event exists in our collection. If not, we return an empty array. No dramas. If there is an event, we loop through each stored callback and pass the data into it. If there are no callbacks (which shouldn’t ever be the case), it’s all good, because we created that event with an empty array in the `subscribe` method.

That’s it for PubSub. Let’s move on to the next part!

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-3)The core Store object

Now that we’ve got our Pub/Sub module, we’ve got our only dependency for the meat‘n’taters of this little application: the Store. We’ll go ahead and start fleshing that out now.

Let’s first outline what this does.

The Store is our central object. Each time you see `@import store from '../lib/store.js`, you'll be pulling in the object that we're going to write. It'll contain a `state` object that, in turn, contains our application state, a `commit` method that will call our **>mutations**, and lastly, a `dispatch` function that will call our **actions**. Amongst this and core to the `Store` object, there will be a Proxy-based system that will monitor and broadcast state changes with our `PubSub` module.

Start off by creating a new directory in your `js` directory called `store`. In there, create a new file called `store.js`. Your `js`directory should now look like this:

```
/js
└── lib
    └── pubsub.js
└──store
    └── store.js
```

Open up `store.js` and import our Pub/Sub module. To do that, add the following right at the top of the file:

```
import PubSub from '../lib/pubsub.js';
```

For those who work with ES6 regularly, this will be very recognizable. Running this sort of code without a bundler will probably be less recognizable though. There's a [heck of a lot of support](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Browser_compatibility) already for this approach, too!

Next, let's start building out our object. Straight after the import, add the following to `store.js`:

```
export default class Store {
  constructor(params) {
    let self = this;
  }
}
```

This is all pretty self-explanatory, so let's add the next bit. We're going to add default objects for `state`, `actions`, and `mutations`. We're also adding a `status` element that we'll use to determine what the object is doing at any given time. This goes right after `let self = this;`:

```
self.actions = {};
self.mutations = {};
self.state = {};
self.status = 'resting';
```

Straight after that, we'll create a new `PubSub` instance that will be attached the `Store` as an `events` element:

```
self.events = new PubSub();
```

Next, we're going to search the passed `params` object to see if any `actions` or `mutations` were passed in. When the `Store` object is instantiated, we can pass in an object of data. Included in that can be a collection of `actions` and `mutations` that control the flow of data in our store. The following code comes next right after the last line that you added:

```
if(params.hasOwnProperty('actions')) {
  self.actions = params.actions;
}

if(params.hasOwnProperty('mutations')) {
  self.mutations = params.mutations;
}
```

That's all of our defaults set and nearly all of our potential params set. Let's take a look at how our `Store` object keeps track of all of the changes. We're going to use a [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to do this. What the Proxy does is essentially work on behalf of our state object. If we add a `get` trap, we can monitor every time that the object is asked for data. Similarly with a `set` trap, we can keep an eye on changes that are made to the object. This is the main part we're interested in today. Add the following straight after the last lines that you added and we'll discuss what it's doing:

```
self.state = new Proxy((params.state || {}), {
  set: function(state, key, value) {

    state[key] = value;

    console.log(`stateChange: ${key}: ${value}`);

    self.events.publish('stateChange', self.state);

    if(self.status !== 'mutation') {
      console.warn(`You should use a mutation to set ${key}`);
    }

    self.status = 'resting';

    return true;
  }
});
```

What's happening here is we're trapping the state object `set`operations. That means that when a mutation runs something like `state.name = 'Foo'` , this trap catches it before it can be set and provides us an opportunity to work with the change or even reject it completely. In our context though, we're setting the change and then logging it to the console. We're then publishing a `stateChange` event with our `PubSub` module. Anything subscribed to that event's callback will be called. Lastly, we're checking the status of `Store`. If it's not currently running a `mutation`, it probably means that the state was updated manually. We add a little warning in the console for that to give the developer a little telling off.

There's a lot going on there, but I hope you're starting to see how this is all coming together and importantly, how we're able to maintain state centrally, thanks to Proxy and Pub/Sub.

#### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-4)Dispatch and commit

Now that we've added our core elements of the `Store`, let's add two methods. One that will call our `actions` named `dispatch` and another that will call our `mutations` called `commit`. Let's start with `dispatch` by adding this method after your `constructor` in `store.js`:

```
dispatch(actionKey, payload) {

  let self = this;

  if(typeof self.actions[actionKey] !== 'function') {
    console.error(`Action "${actionKey} doesn't exist.`);
    return false;
  }

  console.groupCollapsed(`ACTION: ${actionKey}`);

  self.status = 'action';

  self.actions[actionKey](self, payload);

  console.groupEnd();

  return true;
}
```

The process here is: look for an action and, if it exists, set a status and call the action while creating a logging group that keeps all of our logs nice and neat. Anything that is logged (like a mutation or Proxy log) will be kept in the group that we define. If no action is set, it'll log an error and bail. That was pretty straightforward, and the `commit`method is even more straightforward.

Add this after your `dispatch` method:

```
commit(mutationKey, payload) {
  let self = this;

  if(typeof self.mutations[mutationKey] !== 'function') {
    console.log(`Mutation "${mutationKey}" doesn't exist`);
    return false;
  }

  self.status = 'mutation';

  let newState = self.mutations[mutationKey](self.state, payload);

  self.state = Object.assign(self.state, newState);

  return true;
}
```

This method is pretty similar, but let's run through the process anyway. If the mutation can be found, we run it and get our new state from its return value. We then take that new state and merge it with our existing state to create an up-to-date version of our state.

With those methods added, our `Store` object is pretty much complete. You could actually modular-ize this application now if you wanted because we've added most of the bits that we need. You could also add some tests to check that everything run as expected. But I'm not going to leave you hanging like that. Let's make it all actually do what we set out to do and continue with our little app!

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-5)Creating a base component

To communicate with our store, we've got three main areas that update independently based on what's stored in it. We're going to make a list of submitted items, a visual count of those items, and another one that's visually hidden with more accurate information for screen readers. These all do different things, but they would all benefit from something shared to control their local state. We're going to make a base component class!

First up, let's create a file. In the `lib` directory, go ahead and create a file called `component.js`. The path for me is:

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/lib/component.js
```

Once that file is created, open it and add the following:

```
import Store from '../store/store.js';

export default class Component {
  constructor(props = {}) {
    let self = this;

    this.render = this.render || function() {};

    if(props.store instanceof Store) {
      props.store.events.subscribe('stateChange', () => self.render());
    }

    if(props.hasOwnProperty('element')) {
      this.element = props.element;
    }
  }
}
```

Let's talk through this chunk of code. First up, we're importing the `Store` *class*. This isn't because we want an instance of it, but more for checking one of our properties in the `constructor`. Speaking of which, in the `constructor` we're looking to see if we've got a render method. If this `Component` class is the parent of another class, then that will have likely set its own method for `render`. If there is no method set, we create an empty method that will prevent things from breaking.

After this, we do the check against the `Store` class like I mentioned above. We do this to make sure that the `store` prop is a `Store`class instance so we can confidently use its methods and properties. Speaking of which, we're subscribing to the global `stateChange`event so our object can *react*. This is calling the `render` function each time the state changes.

That's all we need to write for that class. It'll be used as a parent class that other components classes will `extend`. Let's crack on with those!

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-6)Creating our components

Like I said earlier, we've got three components to make and their all going to `extend` the base `Component` class. Let's start off with the biggest one: the list of items!

In your `js` directory, create a new folder called `components` and in there create a new file called `list.js`. For me the path is:

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/components/list.js
```

Open up that file and paste this whole chunk of code in there:

```
import Component from '../lib/component.js';
import store from '../store/index.js';

export default class List extends Component {

  constructor() {
    super({
      store,
      element: document.querySelector('.js-items')
    });
  }

  render() {
    let self = this;

    if(store.state.items.length === 0) {
      self.element.innerHTML = `<p class="no-items">You've done nothing yet &#x1f622;</p>`;
      return;
    }

    self.element.innerHTML = `
      <ul class="app__items">
        ${store.state.items.map(item => {
          return `
            <li>${item}<button aria-label="Delete this item">×</button></li>
          `
        }).join('')}
      </ul>
    `;

    self.element.querySelectorAll('button').forEach((button, index) => {
      button.addEventListener('click', () => {
        store.dispatch('clearItem', { index });
      });
    });
  }
};
```

I hope that code is pretty self-explanatory after what we've learned earlier in this tutorial, but let's skim through it anyway. We start off by passing our `Store` instance up to the `Component` parent class that we are extending. This is the `Component` class that we've just written.

After that, we declare our render method that gets called each time the `stateChange` Pub/Sub event happens. In this `render` method we put out either a list of items, or a little notice if there are no items. You'll also notice that each button has an event attached to it and they dispatch and action within our store. This action doesn't exist yet, but we'll get to it soon.

Next up, create two more files. These are two new components, but they're tiny — so we're just going to paste some code in them and move on.

First, create `count.js` in your `component` directory and paste the following in it:

```
import Component from '../lib/component.js';
import store from '../store/index.js';

export default class Count extends Component {
  constructor() {
    super({
      store,
      element: document.querySelector('.js-count')
    });
  }

  render() {
    let suffix = store.state.items.length !== 1 ? 's' : '';
    let emoji = store.state.items.length > 0 ? '&#x1f64c;' : '&#x1f622;';

    this.element.innerHTML = `
      <small>You've done</small>
      ${store.state.items.length}
      <small>thing${suffix} today ${emoji}</small>
    `;
  }
}
```

Looks pretty similar to list, huh? There's nothing in here that we haven't already covered, so let's add another file. In the same `components` directory add a `status.js` file and paste the following in it:

```
import Component from '../lib/component.js';
import store from '../store/index.js';

export default class Status extends Component {
  constructor() {
    super({
      store,
      element: document.querySelector('.js-status')
    });
  }

  render() {
    let self = this;
    let suffix = store.state.items.length !== 1 ? 's' : '';

    self.element.innerHTML = `${store.state.items.length} item${suffix}`;
  }
}
```

Again, we've covered everything in there, but you can see how handy it is having a base `Component` to work with, right? That's one of the many benefits of [Object-orientated Programming](https://en.wikipedia.org/wiki/Object-oriented_programming), which is what most of this tutorial is based on.

Finally, let's check that your `js` directory is looking right. This is the structure of where we're currently at:

```
/src
├── js
│   ├── components
│   │   ├── count.js
│   │   ├── list.js
│   │   └── status.js
│   ├──lib
│   │  ├──component.js
│   │  └──pubsub.js
└───── store
       └──store.js
       └──main.js
```

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-7)Let's wire it up

Now that we've got our front-end components and our main `Store`, all we've got to do is wire it all up.

We've got our store system and the components to render and interact with its data. Let's now wrap up by hooking up the two separate ends of the app and make the whole thing work together. We’ll need to add an initial state, some `actions` and some `mutations`. In your `store` directory, add a new file called `state.js`. For me it's like this:

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/store/state.js
```

Open up that file and add the following:

```
export default {
  items: [
    'I made this',
    'Another thing'
  ]
};
```

This is pretty self-explanatory. We're adding a default set of items so that on first-load, our little app will be fully interactive. Let's move on to some `actions`. In your `store` directory, create a new file called `actions.js` and add the following to it:

```
export default {
  addItem(context, payload) {
    context.commit('addItem', payload);
  },
  clearItem(context, payload) {
    context.commit('clearItem', payload);
  }
};
```

The actions in this app are pretty minimal. Essentially, each action is passing a payload to a mutation, which in turn, commits the data to store. The `context`, as we learned earlier, is the instance of the `Store` class and the `payload` is passed in by whatever dispatches the action. Speaking of mutations, let's add some. In this same directory add a new file called `mutations.js`. Open it up and add the following:

```
export default {
  addItem(state, payload) {
    state.items.push(payload);

    return state;
  },
  clearItem(state, payload) {
    state.items.splice(payload.index, 1);

    return state;
  }
};
```

Like the actions, these mutations are minimal. In my opinion, your mutations should always be simple because they have one job: mutate the store's state. As a result, these examples are as complex as they should ever be. Any proper logic should happen in your `actions`. As you can see for this system, we return the new version of the state so that the `Store`'s <code>commit` method can do its magic and update everything. With that, the main elements of the store system are in place. Let's glue them together with an index file.

In the same directory, create a new file called `index.js`. Open it up and add the following:

```
import actions from './actions.js';
import mutations from './mutations.js';
import state from './state.js';
import Store from './store.js';

export default new Store({
  actions,
  mutations,
  state
});
```

All this file is doing is importing all of our store pieces and glueing them all together as one succinct `Store` instance. Job done!

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-8)The final piece of the puzzle

The last thing we need to put together is the `main.js` file that we included in our `index.html` page *waaaay* up at the start of this tutorial. Once we get this sorted, we'll be able to fire up our browsers and enjoy our hard work! Create a new file called `main.js` at the root of your `js` directory. This is how it looks for me:

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/main.js
```

Open it up and add the following:

```
import store from './store/index.js'; 

import Count from './components/count.js';
import List from './components/list.js';
import Status from './components/status.js';

const formElement = document.querySelector('.js-form');
const inputElement = document.querySelector('#new-item-field');
```

So far, all we're doing is pulling in dependencies that we need. We've got our `Store`, our front-end components and a couple of DOM elements to work with. Let's add this next bit to make the form interactive, straight under that code:

```
formElement.addEventListener('submit', evt => {
  evt.preventDefault();

  let value = inputElement.value.trim();

  if(value.length) {
    store.dispatch('addItem', value);
    inputElement.value = '';
    inputElement.focus();
  }
});
```

What we're doing here is adding an event listener to the form and preventing it from submitting. We then grab the value of the textbox and trim any whitespace off it. We do this because we want to check if there's actually any content to pass to the store next. Finally, if there's content, we dispatch our `addItem` action with that content and let our shiny new `store` deal with it for us.

Let's add some more code to `main.js`. Under the event listener, add the following:

```
const countInstance = new Count();
const listInstance = new List();
const statusInstance = new Status();

countInstance.render();
listInstance.render();
statusInstance.render();
```

All we're doing here is creating new instances of our components and calling each of their `render` methods so that we get our initial state on the page.

With that final addition, we are done!

Open up your browser, refresh and bask in the glory of your new state managed app. Go ahead and add something like *"Finished this awesome tutorial"* in there. Pretty neat, huh?

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-9)Next steps

There's a lot of stuff you could do with this little system that we've put together. Here are some ideas for taking it further on your own:

- You could implement some local storage to maintain state, even when you reload
- You could pull out the front-end of this and have a little state system for your projects
- You could continue to develop the front-end of this app and make it look awesome. (I'd be really interested to see your work, so please share!)
- You could work with some remote data and maybe even an API
- You could take what you've learned about `Proxy` and the Pub/Sub pattern and develop those transferable skills further

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-10)Wrapping up

Thanks for learning about how these state systems work with me. The big, popular ones are much more complex and smarter that what we've done — but it's still useful to get an idea of how these systems work and unravel the mystery behind them. It's also useful to learn how powerful JavaScript can be with no frameworks whatsoever.

If you want a finished version of this little system, check out this [GitHub repository](https://github.com/hankchizljaw/vanilla-js-state-management). You can also see a demo [here](https://vanilla-js-state-management.hankchizljaw.io/).

If you develop on this further, I'd love to see it, so hit me up on [Twitter](https://twitter.com/hankchizljaw) or post in the comments below if you do!