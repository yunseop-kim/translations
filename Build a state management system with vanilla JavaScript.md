# 바닐라 자바스크립트로 상태 관리 시스템 구축

BY [ANDY BELL ](https://css-tricks.com/author/andybell/)ON JULY 25, 2018

상태 관리는 소프트웨어에서 새로운 것이 아니지만, 자바스크립트로 소프트웨어를 구축할 때는 상대적으로 생소하게 느껴질 것입니다. 전통적으로 우리는 DOM 자체에서 상태를 관리했거나, 윈도우 전역 객체에 할당하기도 했습니다. 그러나 이제는 라이브러리와 프레임워크로 이 문제를 해결 할 수 있습니다. Redux, MobX, Vuex 같은 라이브러리들은 컴포넌트 간 상태 관리를 손쉽게 관리할 수 있습니다. 이는 어플리케이션의 탄력성(resilience)에 적합하며 React 또는 Vue와 같은 상태 기반의 반응형 프레임워크에서 잘 동작합니다.

이 라이브러리들은 어떻게 작동할까요? 우리가 스스로 코드를 작성하려면 무엇이 필요할까요? 파헤쳐보면 간단하게 몇 가지 일반적인 패턴을 배우고 우리에게 유용한 API에 대해서 배울 수 있습니다.

시작하기 전에, 당신이 자바스크립트에 대해 중급자 이상의 지식이 있는 것이 좋습니다. 데이터 타입에 대해 알아야 하며, ES6 이상의 자바스크립트 기능에 대한 지식이 있어야 합니다. 그렇지 않다면 [이곳](https://css-tricks.com/learning-gutenberg-4-modern-javascript-syntax/)에서 선행 학습을 해주세요. Redux 나 MobX를 이걸로 대체해야한다는 말은 아닙니다. 우리는 스킬 업을 위한 작은 프로젝트를 진행할 것입니다. JavaScript 페이로드의 크기를 신경쓰고 있다면 작은 애플리케이션에 확실히 도움이 될 수 있습니다.

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-0)시작하기

코드 속으로 빠져들기 전에, 우리가 무엇을 만드는지 확인해 봅시다. "한 일 리스트"는 여러분이 오늘 달성한 것들을 추가시키는 앱입니다. 프레임워크의 의존성 없이 다양한 UI요소를 마술같이 업데이트 할 겁니다. (사실 마술은 아닙니다.) Behind the scenes, we’ve got a little state system that’s sitting, waiting for instructions and maintaining a single source of truth in a predictable fashion.

[데모](https://vanilla-js-state-management.hankchizljaw.io/)

[저장소](http://github.com/hankchizljaw/vanilla-js-state-management)

멋지죠? 관리를 먼저 해 봅시다. 해당 튜토리얼을 잘 유지할 수 있도록 약간의 보일러플레이트를 넣었습니다. 여러분이 해야 할 첫번째 일은 [GitHub에서 클론](https://github.com/hankchizljaw/vanilla-js-state-management-boilerplate) 하거나 [ZIP 아카이브](https://github.com/hankchizljaw/vanilla-js-state-management-boilerplate/archive/master.zip)를 다운로드 한 뒤 압축을 푸는 것입니다.

작업을 마치면 이제 로컬 웹 서버를 실행해야 합니다. 저는 [http-server](https://www.npmjs.com/package/http-server)와 같은 패키지를 이용할 것이지만, 여러분의 입맛에 맞는 다른 서버를 사용해도 됩니다. 로컬로 실행하면 다음과 같은 내용이 표시됩니다:

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

다음으로, `src` 폴더를 연 다음 거기에 있는 js 폴더를 열고, `lib` 이라는 새 폴더를 만듭니다. 그리고 안에 `pubsub.js` 라는 파일을 만듭니다.

`js ` 폴더의 구조는 아래와 같습니다.

```
/js
├── lib
└── pubsub.js
```

작은 [Pub/Sub 패턴](https://msdn.microsoft.com/en-us/magazine/hh201955.aspx) ("Publish/Subscribe"의 줄임말) 을 만들 것이므로 `pubsub.js`를 여십시오. 우리는 어플리케이션의 어느 한 부분을 명명 된 이벤트로 구독(subscribe) 할 수 있는 기능을 만들고 있습니다. 어플리케이션의 다른 부분에서는 연관된 이벤트를 게시(publish) 할수 있습니다.

Pub / Sub는 때로는 이해하기가 쉽지 않으므로 비유는 어떨까요? 당신은 레스토랑에서 일하고 있으며, 고객이 에피타이저와 메인 코스를 주문했다고 상상해보세요. 레스토랑에서 일한 적이 있다면, 서빙하는 사람이 에피타이저를 치울 때, 요리사가 어느 테이블의 에피타이저를 치우는지 알게 됩니다. 이것은 해당 테이블의 메인 코스 요리의 시작을 알리는 단서입니다. 큰 레스토랑에서는 요리사가 몇 명이 있어서 다른 요리를 먹을수도 있습니다. 요리사들은 모두 고객이 에피타이저를 다 먹었다는 단서를 서빙하는 사람에게서 *구독*하므로 메인 코스 요리를 준비하는 *기능*을 수행합니다. 따라서 서로 다른 기능 (콜백)을 수행하기 위해 동일한 단서 (이벤트)를 기다리는 여러 명의 요리사가 있습니다.

![img](https://cdn.css-tricks.com/wp-content/uploads/2018/07/state-management-restaurant.jpg)

이렇게 생각하면 도움이 될 것이라 생각합니다. 계속 가시죠!

Pub/Sub 패턴은 모든 구독(subscribe)을 돌며 해당 페이로드로 콜백을 발생시킵니다. 이는 앱에 대한 매우 우아한 반응형 흐름을 만드는 좋은 방법이며 몇 줄의 코드만으로도 해결할 수 있습니다.

 `pubsub.js`에 다음을 추가합니다.

```
export default class PubSub {
  constructor() {
    this.events = {};
  }
}
```

 새로운 클래스를 선언하고, 기본적으로 `this.events` 를 빈 오브젝트로 설정합니다. `this.events` 객체는 명명 된 이벤트를 담습니다. constructor의 닫는 괄호 뒤에 다음을 추가합니다.

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

이것이 우리의 구독(subscribe) 메소드입니다. 이벤트의 유일한 이름인 `event`와 콜백 함수를 파라미터로 전달합니다. `events` 콜렉션에 혹시 일치하는 이벤트가 없다면, 빈 배열로 생성하여 나중에 다시 입력 할 필요가 없도록 합니다. 그런 다음 콜백을 콜렉션으로 푸시합니다. 그것이 이미 존재한다면, 이것은 모든 메소드가 할 것입니다. 우리는 이벤트 콜렉션의 길이를 반환합니다. 누구든지 쉽게 이벤트가 얼마나 있는지 알 수 있게 하기 위해서입니다.

이제 구독(subscribe) 메소드을 알았으니 다음에 무엇을 구현할지 생각해보십시오. 여러분은 `publish` 메소드를 생성하리라는 것을 예상하셨을 겁니다. subscribe 메소드 다음에 다음을 추가하십시오.

```
publish(event, data = {}) {

  let self = this;

  if(!self.events.hasOwnProperty(event)) {
    return [];
  }

  return self.events[event].map(callback => callback(data));
}
```

이 메소드는 먼저 전달 된 이벤트가 컬렉션에 있는지 확인합니다. 그렇지 않으면 빈 배열을 반환합니다. 특별하진 않습니다. 이벤트가 있는 경우, 저장된 콜백을 반복하여 데이터를 전달합니다. If there are no callbacks (which shouldn’t ever be the case), it’s all good, because we created that event with an empty array in the `subscribe` method.

그것은 PubSub을 위한 것입니다. 다음 파트로 넘어 갑시다!

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-3)코어 스토어 객체

Now that we’ve got our Pub/Sub module, we’ve got our only dependency for the meat‘n’taters of this little application: the Store. We’ll go ahead and start fleshing that out now.

먼저 무엇을 하는지 개괄을 설명하겠습니다.

스토어는 우리의 중심 객체입니다. `@import store from '../lib/store.js`를 볼 때마다, 우리는 작성하려고 하는 객체를 가져올 것입니다. 그것은 `state` 객체를 포함 할 것이고, 우리의 애플리케이션 상태, **mutations**를 호출 할 `commit` 메소드, 마지막으로 **actions**를 호출 할 `dispatch` 메소드를 포함 할 것입니다. 이 객체와 `Store` 객체의 핵심에는 `PubSub` 모듈을 사용하여 상태 변경을 모니터링하고 브로드캐스팅하는 프록시 기반 시스템이 있습니다.

`js` 디렉토리에`store`라고하는 새로운 디렉토리를 만듭니다. 거기에`store.js`라는 새로운 파일을 만듭니다. `js` 디렉토리는 다음과 같을 것입니다.

```
/js
└── lib
    └── pubsub.js
└──store
    └── store.js
```

`store.js`를 열고 Pub / Sub 모듈을 가져옵니다. 그렇게하려면 파일의 맨 위에 다음과 같이 추가하십시오.

```
import PubSub from '../lib/pubsub.js';
```

자주 ES6을 사용하는 사람들에게 이것은 매우 잘 알려져 있습니다. bundler없이 이런 종류의 코드를 실행하는 것은 알아볼 수있을 것입니다. 이미 이 접근법에 대한 [많은 지원](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Browser_compatibility)이 이미 있습니다!

다음으로 객체를 만들기 시작합시다. import 후 `store.js`에 다음을 추가하십시오 :

```
export default class Store {
  constructor(params) {
    let self = this;
  }
}
```

This is all pretty self-explanatory, so let's add the next bit. We're going to add default objects for `state`, `actions`, and `mutations`. We're also adding a `status` element that we'll use to determine what the object is doing at any given time. This goes right after `let self = this;`:

이것은 모두 어렵지 않게 이해할 수 있을 것입니다. 다음 문장을 추가합니다. 우리는 `state`, `actions`, `mutations` 에 기본 객체를 추가 할 것입니다. 우리는 또한 주어진 시간에 객체가 무엇을하고 있는지를 결정하는데 사용할`status` 요소를 추가하고 있습니다. 이것은 `let self = this;` 직후에 나옵니다.

```
self.actions = {};
self.mutations = {};
self.state = {};
self.status = 'resting';
```

이어서 우리는 `Store`를  `events` 엘리먼트로 붙일 새로운 `PubSub` 인스턴스를 생성 할 것입니다.

```
self.events = new PubSub();
```

다음으로 전달 된 `params` 객체를 검색하여 `actions` 또는 `mutations`가 전달되었는지 확인합니다. Store 객체가 인스턴스화되면 데이터 객체를 전달할 수 있습니다. 여기에는 우리 스토어의 데이터 흐름을 제어하는`actions`와`mutations` 컬렉션이 포함될 수 있습니다. 다음 코드는 아까 추가 한 마지막 줄 바로 다음에 옵니다.

```
if(params.hasOwnProperty('actions')) {
  self.actions = params.actions;
}

if(params.hasOwnProperty('mutations')) {
  self.mutations = params.mutations;
}
```

이것이 우리의 모든 기본 설정이며 거의 모든 잠재적 매개 변수가 설정됩니다. `Store` 객체가 모든 변화를 추적하는 방법을 살펴 보겠습니다. 우리는 이를 위해 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)를 사용할 것입니다. 프록시가 하는 일은 근본적으로 우리 주 객체 대신에 작동합니다. `get` 트랩을 추가하면 객체가 데이터를 요청할 때마다 모니터링 할 수 있습니다. 마찬가지로 `set` 트랩과 마찬가지로 우리는 객체에 대한 변경을 감시 할 수 있습니다. 이것이 우리가 관심있는 주요한 부분입니다. 추가 한 마지막 줄 다음에 다음 내용을 추가하십시오.

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

What's happening here is we're trapping the state object `set`operations. That means that when a `mutation` runs something like `state.name = 'Foo'` , this trap catches it before it can be set and provides us an opportunity to work with the change or even reject it completely. In our context though, we're setting the change and then logging it to the console. We're then publishing a `stateChange` event with our `PubSub` module. Anything subscribed to that event's callback will be called. Lastly, we're checking the status of `Store`. If it's not currently running a `mutation`, it probably means that the state was updated manually. We add a little warning in the console for that to give the developer a little telling off.

여기서 일어나고있는 것은 상태 객체 `set` 연산을 트래핑하고 있다는 것입니다. 즉,`mutation`이 `state.name ='Foo'`와 같이 실행되면, 이 트랩은 설정되기 전에 그것을 캐치해서 우리가 변화에 대해 작업하거나 완전히 거부 할 수있는 기회를 제공한다는 것을 의미합니다. 하지만 우리는 `context`에서 변경 사항을 설정 한 다음 콘솔에 기록합니다. 우리는 `PubSub` 모듈로`stateChange` 이벤트를 퍼블리싱 할 것입니다. 해당 이벤트의 콜백에 가입 된 모든 항목이 호출됩니다. 마지막으로 Store의 상태를 확인하고 있습니다. 현재 `mutations`가 실행 중이 아니면 상태가 수동으로 업데이트 되었음을 의미합니다. 우리는 개발자에게 약간의 경고를 주기 위해 콘솔에 약간의 경고를 추가합니다.

There's a lot going on there, but I hope you're starting to see how this is all coming together and importantly, how we're able to maintain state centrally, thanks to Proxy and Pub/Sub.

많은 일들이 벌어지고 있지만 이것들이 어떻게 서로 긴밀하게 돌아가고 있는지, 프록시와 Pub/Sub 덕분에 어떻게 상태를 유지할 수 있는지를 알기를 바랍니다.

#### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-4)Dispatch 와 commit

Now that we've added our core elements of the `Store`, let's add two methods. One that will call our `actions` named `dispatch` and another that will call our `mutations` called `commit`. Let's start with `dispatch` by adding this method after your `constructor` in `store.js`:

이제 Store의 코어 요소를 추가 했으므로 두 가지 메소드를 추가하겠습니다. `dispatch`이라는 이름의 `actions`과 `commit`이라 불리는 `mutations` 입니다. `store.js` 에 `constructor` 뒤에 이 `dispatch` 메소드를 추가합시다 :

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

이 프로세스는 다음과 같습니다 :`action`을 찾고, 존재한다면, 상태를 설정하고 모든 로그를 멋지게 유지하는 로깅 그룹을 생성하면서 액션을 호출합니다. (mutation이나 Proxy 로그와 같이) 기록 된 것은 우리가 정의한 그룹에 보관 될 것입니다. If no `action` is set, it'll log an error and bail. 그것은 매우 직관적이고, `commit` 메소드는 훨씬 더 직관적입니다.

`dispatch` 메소드 다음에 이 코드를 추가하십시오 :

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

이 메소드는 꽤 유사하지만 어쨌든 프로세스를 실행 해 봅시다. `mutation`이 발견되면, 그것을 실행하고 리턴 값으로부터 새로운 state를 얻습니다. 그런 다음 새로운 state를 가져와 기존 state와 합쳐 state의 최신 버전을 만듭니다.

추가 된 메서드를 사용하면 Store 객체가 거의 완성됩니다. 우리가 필요로하는 대부분의 코드를 추가 했으므로 원하는 경우 이 애플리케이션을 실제로 모듈화 할 수 있습니다. 모든 테스트가 예상대로 실행되는지 확인하기 위해 몇 가지 테스트를 추가 할 수도 있습니다. 그러나 이 글에서는 다루지 않겠습니다. 계속 우리의 작은 앱을 구현해 봅시다.

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-5) 기초 컴포넌트 생성

To communicate with our store, we've got three main areas that update independently based on what's stored in it. We're going to make a list of submitted items, a visual count of those items, and another one that's visually hidden with more accurate information for screen readers. These all do different things, but they would all benefit from something shared to control their local state. We're going to make a base component class!

First up, let's create a file. In the `lib` directory, go ahead and create a file called `component.js`. The path for me is:

스토어와 주고받기 위해, 저장되는 내용을 기반으로 독립적으로 업데이트되는 세 가지 주요 영역이 있습니다. 제출 된 항목의 목록, 해당 항목의 시각적 개수 및 화면 판독기에 대한보다 정확한 정보로 시각적으로 숨겨진 다른 항목을 만들 것입니다. 이것들은 모두 다른 일을 하지만, 지역 상태(local state)를 통제하기 위해 공유되는 어떤 것에서도 베네핏을 얻습니다. 우리는 기초 컴포넌트 클래스를 만들 것입니다!

먼저 파일을 만듭니다. `lib` 디렉토리에서`component.js` 파일을 생성하십시오. 제 파일의 경로는 다음과 같습니다.

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/lib/component.js
```

Once that file is created, open it and add the following:

파일이 생성되면 파일을 열고 다음을 추가하십시오.

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

의 코드의 덩어리를 파헤쳐 봅시다. 먼저 `Store` *클래스*를 가져옵니다. 이것은 우리가 인스턴스를 원하기 때문이 아니라 `constructor`에서 우리의 프로퍼티 중 하나를 확인하기 위해서입니다. 말하자면, `constructor`에서 렌더링 메소드가 있는지 살펴볼 것입니다. 이 `Component` 클래스가 다른 클래스의 부모라면 `render`를 위한 자체 메소드를 설정했을 것입니다. 메소드가 설정되어 있지 않은 경우, 메소드가 파괴되지 않게하는 빈 메소드를 작성합니다.

After this, we do the check against the `Store` class like I mentioned above. We do this to make sure that the `store` prop is a `Store`class instance so we can confidently use its methods and properties. Speaking of which, we're subscribing to the global `stateChange`event so our object can *react*. This is calling the `render` function each time the state changes.

이 후, 우리는 위에서 언급 한 `Store` 클래스에 대한 검사를 합니다. 우리는`store` 프로퍼티가 `Store` 클래스의 인스턴스이므로 자신의 메소드와 속성을 사용할 수 있습니다. 말하자면, 우리는 전역 `stateChange` 이벤트를 구독하고 있으므로 우리의 객체는 *반응*할 수 있습니다. 이것은 상태가 바뀔 때마다 `render` 함수를 호출하고 있습니다.

That's all we need to write for that class. It'll be used as a parent class that other components classes will `extend`. Let's crack on with those!

그것이 우리가 그 클래스을 위해 작성해야 할 전부입니다.이 클래스는 다른 구성 요소 클래스가 `extend`하는 상위 클래스로 사용됩니다. 한번 작살내 봅시다!

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-6)컴포넌트 만들기

Like I said earlier, we've got three components to make and their all going to `extend` the base `Component` class. Let's start off with the biggest one: the list of items!

In your `js` directory, create a new folder called `components` and in there create a new file called `list.js`. For me the path is:

이전에 말했듯이, 우리는 세 가지 컴포넌트를 만들었고, 그것들은 모두 기본 `Component` 클래스를 `extend`할 것이다. 가장 큰 것부터 시작합시다 : 아이템 목록!

`js` 디렉토리에 `components`라는 새로운 폴더를 만들고 `list.js`라는 새로운 파일을 만듭니다. 제 경로는 다음과 같습니다.

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/components/list.js
```

Open up that file and paste this whole chunk of code in there:

해당 파일을 열고 이 코드 전체를 여기에 붙여 넣으십시오.

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

이 튜토리얼을 배우고 난 후에 저 코드가 아주 쉽게 이해할 수 있게 되기를 바랍니다. 하지만 일단 계속 진행해봅시다. 우리는`Store` 인스턴스를 우리가 `extend` 하고있는 `Component` 부모 클래스까지 전달함으로써 시작합니다. 이것은 방금 작성한`Component` 클래스입니다.

그 다음으로는, render 메소드를 선언합니다. render 메소드는 Pub/Sub의 `stateChange` 이벤트가 발생할 때마다 호출됩니다. `render` 메소드에서는 리스트의 항목을 출력하거나, 항목이 없다면 간단한 메시지를 출력합니다.

또한 각 버튼에는 이벤트가 첨부되어 있으며 `store`내에 `dispatch`과 `action`이 있음을 알 수 있습니다. 이 작업은 아직 존재하지 않지만 곧 진행할 예정입니다.

다음으로 두개의 파일을 더 만듭니다. 이것들은 두 가지 새로운 컴포넌트이지만, 크기는 작습니다. 그래서 우리는 코드를 붙여넣고 계속 진행할 것입니다.

먼저,`component` 디렉토리에`count.js`를 만들고 그 안에 다음을 붙여 넣으십시오.

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

list 코드와 비슷해 보이죠? 여기서는 내용을 다 다루었으므로 다른 파일을 추가해 보겠습니다. 같은 `components` 디렉토리에 `status.js` 파일을 추가하고 그 안에 다음을 붙여 넣으십시오 :

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

다시 말하지만, 우리가 내용을 다 다루었지만, 기본 `Component`가 얼마나 유용하게 사용되는지 알 수 있죠? 이것은 [Object-Oriented Programming](https://en.wikipedia.org/wiki/Object-oriented_programming)의 많은 장점 중 하나입니다. 본 튜토리얼의 대부분은 이 튜토리얼을 기반으로 합니다.

마지막으로`js` 디렉토리가 올바르게 보이는지 확인해 봅시다. 이것은 현재 디렉토리 구조입니다.

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

이제 프론트 엔드 컴포넌트와 메인 `Store` 가 생겼으므로, 요소들을 연결지어 봅시다.

매장 시스템과 구성 요소를 렌더링하고 데이터를 주고받을수 있게 했습니다. 이제 앱의 두 개의 양 끝을 연결하여 모든 것이 돌아가도록 해보겠습니다. 우리는 초기 상태, 일부`actions`과`mutations`를 추가 할 필요가 있습니다. `store` 디렉토리에`state.js`라는 새로운 파일을 추가하십시오. 제 경로는 다음과 같습니다.

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/store/state.js
```

해당 파일을 열고 다음을 추가하십시오.

```
export default {
  items: [
    'I made this',
    'Another thing'
  ]
};
```

어렵지 않게 이해할 수 있을 겁니다. 처음 로드할 때 앱이 온전히 상호작용 할 수 있도록 디폴트 항목을 추가하겠습니다. `actions`으로 넘어 갑시다. `store` 디렉토리에 `actions.js` 라는 새로운 파일을 생성하고 다음 내용을 추가하십시오.

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

이 앱의 동작은 매우 간단합니다. 본질적으로, 각 액션은 `payload` 를 `mutation` 에 전달하고 데이터를 `store` 로 `commit`합니다. 우리가 이전에 배웠던`context`는`Store` 클래스의 인스턴스이고`payload`는 어떤 액션을 보내든간에 전달됩니다. 약간의 코드를 추가해서, mutation에 대해 이야기해 봅시다. 같은 디렉토리에 `mutations.js`라는 새로운 파일을 추가하고 다음 코드를 추가하십시오.

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

action과 마찬가지로 이러한 mutation은 작습니다. 제 의견으로는 mutation은 항상 한 가지 일을 하기 때문에 항상 단순해야합니다. 스토어의 상태를 변경(mutate)하십시오. 결과적으로이 예제들은 지금까지와 마찬가지로 복잡합니다. 당신의`action`에서 적절한 로직이 이루어져야합니다. 이 시스템에서 알 수 있듯이 `Store`의 `<code> commit` 메소드가 모든 것을 수행하고 모든 것을 업데이트 할 수 있도록 새 버전의 상태를 리턴합니다. 이를 통해 store 시스템의 주요 요소가 각 자리에 있습니다. 그것들을 인덱스 파일과 함께 붙이도록 합시다.

동일한 디렉토리에서`index.js`라는 새로운 파일을 만듭니다. 그것을 열고 다음을 추가하십시오 :

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

이 파일이 수행하는 모든 작업은 모든 매장 조각을 가져 와서 하나의 간결한 `Store` 인스턴스로 모두 붙이는 것입니다. 작업 완료!

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-8) 마지막 퍼즐 한 조각

마지막으로 우리가 정리해야 할 것은 이 튜토리얼의 시작 부분에 있는`index.html` 페이지에 포함 된 `main.js` 파일입니다. 이렇게 한번만 정렬하면, 브라우저에서 실행 가능하고 우리가 열심히 작성한 코드가 동작하는 것을 볼 수 있습니다! `js` 디렉토리의 루트에 `main.js`라는 새로운 파일을 만듭니다. 제 경로는 다음과 같습니다.

```
~/Documents/Projects/vanilla-js-state-management-boilerplate/src/js/main.js
```

파일을 열고 다음을 추가하십시오.

```
import store from './store/index.js'; 

import Count from './components/count.js';
import List from './components/list.js';
import Status from './components/status.js';

const formElement = document.querySelector('.js-form');
const inputElement = document.querySelector('#new-item-field');
```

So far, all we're doing is pulling in dependencies that we need. We've got our `Store`, our front-end components and a couple of DOM elements to work with. 다음 코드를 추가하여 해당 코드를 통해 폼을 인터렉티브하게 만들어 봅시다.

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

여기서 우리가하는 일은 이벤트 리스너를 폼에 추가하여 submit하지 못하게 하는 것입니다. 그런 다음 텍스트 상자의 값을 가져 와서 공백을 제거합니다. 실제로 store에 전달할 내용이 있는지 여부를 확인하기 위해 이 작업을 수행합니다. 마지막으로, 콘텐츠가 있다면, 우리는`addItem` 액션을 콘텐츠와 함께 보내고 새로운`store`가 처리하도록합니다.

`main.js`에 좀 더 많은 코드를 추가합시다. 이벤트 리스너에서 다음을 추가하십시오.

```
const countInstance = new Count();
const listInstance = new List();
const statusInstance = new Status();

countInstance.render();
listInstance.render();
statusInstance.render();
```

여기서 우리가 하는 일은 컴포넌트의 새로운 인스턴스를 생성하고 각각의`render` 메소드를 호출하여 페이지의 초기 상태를 얻는 것입니다.

이것을 마지막으로 앱이 완성되었습니다!

브라우저를 켜고, 우리가 만든 상태 관리 앱을 맘껏 작동시켜 보세요. "대빵 멋진 튜토리얼을 끝냄" 같은 항목도 추가해보세요. 멋지죠?

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-9) 다음 단계

우리가 함께 만든 이 작은 시스템으로 할 수 있는 많은 것들이 있습니다. 다음과 같이 스스로 생각해보십시오.

- 다시 로드 할 때도 상태를 유지하기 위해 로컬 저장소를 구현할 수 있습니다.
- 당신의 프로젝트를 위한 작은 상태 시스템을 가질 수 있습니다.
- 당신은 이 어플리케이션의 프론트엔드를 계속 개발하고 멋지게 만들 수 있습니다. (나는 여러분의 작업에 매우 흥미가 있습니다. 만들고 공유해주세요!)
- 일부 원격 데이터와 API로 작업 할 수도 있습니다.
- 당신은`Proxy`와 Pub / Sub 패턴에 대해 배운 것을 받아 들일 수 있고, 이 기술을 응용해서 개발 할 수 있습니다.

### [#](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/#article-header-id-10) 마무리

이 상태 관리 시스템이 어떻게 작동하는지에 대해 학습해 주셔서 감사합니다. 크고 인기있는 시스템들은 우리가 한 작업보다 훨씬 복잡하고 정교합니다. 그러나 이 시스템이 어떻게 작동하는지 생각해보고 그 뒤에 숨겨진 미스테리한 부분을 이해하는 것은 매우 유용한 일입니다. JavaScript가 프레임워크 없이 얼마나 강력한지를 배우는 것도 유용합니다.

이 작은 시스템의 완성 된 버전을 원한다면 이 [GitHub 저장소](https://github.com/hankchizljaw/vanilla-js-state-management)를 확인하십시오. [데모](https://vanilla-js-state-management.hankchizljaw.io/)도 볼 수 있습니다.

만약 이 글에 추가적인 의견이 있으시다면  [트위터](https://twitter.com/hankchizljaw) 나 아래에 댓글을 달아 주세요!