# The Best Explanation of JavaScript Reactivity 🎆

Many front-end JavaScript frameworks (Ex. Angular, React, and Vue) have their own Reactivity engines. By understanding what reactivity is and how it works, you can improve your development skills and more effectively use JavaScript frameworks. In the video and the article below, we build the same sort of Reactivity you see in the Vue source code.

많은 프론트엔드 자바스크립트 프레임워크 (예 : Angular, React, and Vue)에는 자체 반응형 엔진이 있습니다. 반응형의 작동 원리를 이해하면 개발 기술을 향상시킬 수 있고 JavaScript 프레임워크를보다 효과적으로 사용 할 수 있습니다. 아래의 영상과 아티클에서는 Vue 소스 코드에서 볼 수 있는 것과 동일한 종류의 반응형(Reactivity)을 빌드합니다.

*If you watch this video instead of reading the article, watch the* [*next video in the series*](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/) *discussing reactivity and proxies with Evan You, the creator of Vue.*

*기사를 읽는 대신이 비디오를 보는 경우* [*이 시리즈 영상*](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/) 을 보세요. *Vue의 창시자 인 Evan You와 반응형 및 프록시를 논의합니다.*

### 💡 반응형 시스템

Vue’s reactivity system can look like magic when you see it working for the first time. Take this simple Vue app:

Vue의 반응형 시스템은 처음 작동시킬 때 마술처럼 보일 수 있습니다. 이 Vue 앱을 살펴봅시다.

![img](https://cdn-images-1.medium.com/max/1600/1*aLjr0oQBzX7PoUF6oL7yfQ.png)

![img](https://cdn-images-1.medium.com/max/1600/1*neR2Y-0zJseWT8oY1JukXA.png)

Somehow Vue just knows that if `price` changes, it should do three things:

- Update the `price` value on our webpage.
- Recalculate the expression that multiplies `price` `*` `quantity`, and update the page.
- Call the `totalPriceWithTax` function again and update the page.

But wait, I hear you wonder, how does Vue know what to update when the `price` changes, and how does it keep track of everything?

Vue는`price` 값이 바뀌면 세 가지 작업을 하게 됩니다.

- 웹 페이지에서 `price`값을 업데이트
- `price` `*` `quantity`를 곱하는 표현식을 다시 계산하고 페이지를 업데이트
- `totalPriceWithTax` 함수를 다시 호출하고 페이지를 업데이트

그러나 잠깐, 궁금증이 생기실텐데요, `price`가 바뀌면 Vue는 무엇을 업데이트해야 하는지를 어떻게 알 수 있으며, 어떻게 모든 것을 추적 할 수 있을까요?

![img](https://cdn-images-1.medium.com/max/1600/1*t8enMn6h0gjY6HNKoSVC1g.jpeg)

**This is not how JavaScript programming usually works**

If it’s not obvious to you, the big problem we have to address is that programming usually doesn’t work this way. For example, if I run this code:

**이것은 자바 스크립트 프로그래밍이 정상적으로 작동하는 방식이 아닙니다**

이게 당신에게 명확하지 않다면, 우리가 다루어야 할 큰 요점은 프로그래밍이 대개 이런 식으로 작동하지 않는다는 것입니다. 예를 들어서 아래 코드를 실행하면 다음과 같습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*RrDCv_fUYnOl34Eq0afgYw.png)

What do you think it’s going to print? Since we’re not using Vue, it’s going to print `10`.

어떤 값이 나올거 같나요? Vue를 사용하는것이 아니므로 `10`을 나타낼 겁니다.

![img](https://cdn-images-1.medium.com/max/1600/1*q7nHV9seYErboH1DDiEdUg.png)

In Vue we want `total` to get updated whenever `price` or `quantity` get updated. We want:

Vue에서 `total` 또는 `quantity`가 업데이트 될 때마다 `total`이 업데이트 되기를 원합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*aFGF-Go7ONnOtWjfyzytig.png)

Unfortunately, JavaScript is procedural, not reactive, so this doesn’t work in real life. In order to make `total` reactive, we have to use JavaScript to make things behave differently.

안타깝게도 JavaScript는 절차적이지 반응형이 아니므로 실제로는 동작하지 않습니다. `total`을 반응형으로 만들기 위해서 JavaScript를 사용하여 다르게 작동하게 합니다.

### ⚠️ Problem

We need to save how we’re calculating the `total`, so we can re-run it when`price` or `quantity` changes.

### ⚠️ 문제

`total`을 계산해야하기 때문에 `price` 또는`quantity`가 변경 될 때 재실행해야 한다.

### ✅ Solution

First off, we need some way to tell our application, “The code I’m about to run, **store this**, I may need you to run it at another time.” Then we’ll want to run the code, and if `price` or `quantity` variables get updated, run the stored code again.

### ✅ 솔루션

우선적으로는 앱에 알릴 수 있는 방법이 필요합니다. "내가 실행하려고 하는 코드를 **저장해 두었다가** 다른 때 실행할 수도 있습니다." 그러면 원할때 코드를 실행 할 수 있겠죠. `price` 또는`quantity` 변수가 업데이트 되면, 저장된 코드를 다시 실행하게 됩니다.

![img](https://cdn-images-1.medium.com/max/1600/0*Nh-FQoHiDHncmQSi.png)

We might do this by recording the function so we can run it again.

함수를 다시 실행하기 위해서 이 함수를 기록해 둡니다.

![img](https://cdn-images-1.medium.com/max/1600/1*vD19ImKAK2WYySJGIvrKtQ.png)

Notice that we store an anonymous function inside the `target` variable, and then call a `record` function. Using the ES6 arrow syntax I could also write this as:

익명의 함수를`target` 변수 안에 저장하고,`record` 함수를 호출합니다. ES6 화살표 구문을 사용하여 다음과 같이 작성할 수도 있습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*z0E_bw1_XBjGdM0ab_pyDg.png)

The definition of the `record` is simply:

`record`의 정의는 간단합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*6tRbHmwr7mzy5CNemhkTcg.png)

We’re storing the `target` (in our case the `{` `total = price * quantity` `}`) so we can run it later, perhaps with a `replay` function that runs all the things we’ve recorded.

우리는 `target`을 저장하고 있습니다. (해당 코드의 경우 `{total = price * quantity)`). 우리가 기록한 모든 것을 실행하는`replay` 함수를 나중에 실행할 수 있습니다. .

![img](https://cdn-images-1.medium.com/max/1600/1*dyxkUSFl1S3m4dFCgc2jZw.png)

This goes through all the anonymous functions we have stored inside the storage array and executes each of them.

Then in our code, we can just:

이것은 스토리지 배열 내부에 저장된 모든 익명 함수을 거쳐 각각을 실행합니다.

그러면 코드에서 다음과 같이 할 수 있습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*Fr8Oif-PkvmyFDKDt5Xm6w.png)

Simple enough, right? Here’s the code in it’s entirety if you need to read through and try to grasp it one more time. FYI, I am coding this in a particular way, in case you’re wondering why.

간단하죠? 아래는 전체 코드입니다. 처음부터 끝까지 읽고 다시한번 이해하고자 하는 경우 참고하세요. 참고로, 이유를 궁금해하는 경우에 대비하여 특정 방식으로 코딩하고 있습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*TpEBEstjfM4FNMYuh37BLA.png)

![img](https://cdn-images-1.medium.com/max/1600/0*0a_165xKF15xL889.png)

### ⚠️ Problem

We could go on recording targets as needed, but it’d be nice to have a more robust solution that will scale with our app. Perhaps a class that takes care of maintaining a list of targets that get notified when we need them to get re-run.

### ⚠️ 문제

필요에 따라서 기록할 타겟을 실행할 수도 있지만, 앱으로 확장할 수 있는 더 강력한 솔루션을 갖는게 좋을 것입니다. 아마도 재실행을 원할 때, 알림을 받는 타겟 목록을 유지 관리하는 클래스일 것입니다.

### ✅ Solution: A Dependency Class

One way we can begin to solve this problem is by encapsulating this behavior into its own class, a **Dependency Class** which implements the standard programming observer pattern.

So, if we create a JavaScript class to manage our dependencies (which is closer to how Vue handles things), it might look like this:

### ✅ 솔루션 : 종속성 클래스

이 문제를 해결하기 위해 시작할 수 있는 한 가지 방법은 자체 클래스에 이 동작을 캡슐화 함으로써, 관찰자 패턴(observer pattern)을 **종속 클래스**로 구현하는 것입니다.

따라서 종속성 (Vue가 일을 처리하는 방법에 더 가깝습니다.)을 관리하기 위한 JavaScript 클래스를 만든다면 다음과 같을 것입니다.

![img](https://cdn-images-1.medium.com/max/1600/1*9NnQmGxZfmxhRJBUEs4Z7g.png)

Notice instead of `storage` we’re now storing our anonymous functions in `subscribers`. Instead of our `record` function we now call `depend` and we now use `notify` instead of `replay`. To get this running:

`storage` 대신에 익명 함수를 `subscribers`에 저장하고 있습니다. `record` 함수 대신에`depend`을 호출하고`replay` 대신`notify`를 사용합니다. 이를 실행하려면 다음을 코드를 작성하세요.

![img](https://cdn-images-1.medium.com/max/1600/1*Y5XJpipq7-Po1mP_eJoCGw.png)

It still works, and now our code feels more reusable. Only thing that still feels a little weird is the setting and running of the `target`.

여전히 작동하며 이제 코드가 더 재사용 될 수 있습니다. 조금 이상한 느낌을 주는 것은 `target` 설정과 실행뿐입니다.

### ⚠️ Problem

In the future we’re going to have a Dep class for each variable, and it’ll be nice to encapsulate the behavior of creating anonymous functions that need to be watched for updates. Perhaps a `watcher` function might be in order to take care of this behavior.

So instead of calling:

### ⚠️ 문제

앞으로는 각 변수에 대해 Dep 클래스를 갖게 될 것이며, 업데이트가 필요한 익명 함수를 만드는 동작을 캡슐화하는 것이 좋습니다. 아마도 `감시자 (watcher)` 기능이 이러한 행동을 처리할 겁니다.

아래 코드를 호출하는 대신에,

![img](https://cdn-images-1.medium.com/max/1600/1*mo9tPOcAy-qC1VZ6Bnz76A.png)

(this is just the code from above)

We can instead just call:

(위에서 작성한 코드와 같습니다.)

대신 다음을 호출하도록 합시다.

![img](https://cdn-images-1.medium.com/max/1600/1*2TPYfKaV4UBEReN8PWwzbQ.png)

### ✅ Solution: A Watcher Function

Inside our Watcher function we can do a few simple things:

### ✅ 솔루션 : 감시자 (Watcher) 함수

Watcher 함수에서 우리는 몇 가지 간단한 일을 할 수 있습니다 :

![img](https://cdn-images-1.medium.com/max/1600/1*U7bJcE5Ad7lxbQUP-U68uw.png)

As you can see, the `watcher` function takes a `myFunc` argument, sets that as a our global `target` property, calls `dep.depend()` to add our target as a subscriber, calls the `target` function, and resets the `target`.

Now when we run the following:

보시다시피,`watcher` 함수는`myFunc` 인수로 취해서  전역 `target` 프로퍼티로 설정하고 `dep.depend()`를 호출하여 타겟을 구독자(subscriber)로 추가하고`target` 함수를 호출합니다 ,`target`을 리셋합니다.

아래를 실행할 때 결과입니다.

![img](https://cdn-images-1.medium.com/max/1600/1*vefCUnWyacq0GxC3TRI0Cw.png)

![img](https://cdn-images-1.medium.com/max/1600/0*D05AHN0_GUXoMVM8.png)

You might be wondering why we implemented `target` as a global variable, rather than passing it into our functions where needed. There is a good reason for this, which will become obvious by the end of our article.

`target`을 전역 변수로 구현 한 이유가 궁금하실겁니다. 이것에 대한 충분한 이유가 있으며, 이는 우리 기사의 끝 부분에서 명확해질 것입니다.

### ⚠️ Problem

We have a single `Dep class`, but what we really want is each of our variables to have its own Dep. Let me move things into properties before we go any further.

### ⚠️ 문제

우리는 하나의`Dep 클래스 '를 가지고 있지만, 우리가 정말로 원하는 것은 각각의 변수가 자신의 Dep를 갖기를 원한다는 것입니다. Let me move things into properties before we go any further.

![img](https://cdn-images-1.medium.com/max/1600/1*YBknbJTkI-za0L9eMAFayQ.png)

Let’s assume for a minute that each of our properties (`price` and `quantity`) have their own internal Dep class.

우리의 각 프로퍼티 (`price` 와 `quantity`)가 그들 자신의 내부 Dep 클래스를 가지고 있다고 가정해 봅시다.

![img](https://cdn-images-1.medium.com/max/1600/0*kV4iCRoguwO5C_JQ.png)

Now when we run:

아래 코드를 실행시킵시다.

![img](https://cdn-images-1.medium.com/max/1600/1*-rznzvwxr5clvYdPVq2MfA.png)

Since the `data.price` value is accessed (which it is), I want the `price` property’s Dep class to push our anonymous function (stored in `target`) onto its subscriber array (by calling `dep.depend()`). Since `data.quantity` is accessed I also want the `quantity` property Dep class to push this anonymous function (stored in `target`) into its subscriber array.

`data.price` 값에 접근했으므로 `price` 속성의 Dep 클래스가 `target`에 저장되어 있는 익명 함수를 subscriber 배열에 push하고자 합니다(`dep.depend()` 를 호출함으로서). `data.quantity`에 접근했으므로 `amount` 속성 Dep 클래스가 이 `target`에 저장되어 있는 익명 함수를 subscriber 배열에 push 하고자 합니다.

![img](https://cdn-images-1.medium.com/max/1600/0*E-_YXfn3vJe7S_Ry.png)

If I have another anonymous function where just `data.price` is accessed, I want that pushed just to the `price` property Dep class.

`data.price` 만 액세스하는 또 다른 익명의 함수가 있으면 `price` 속성 Dep 클래스에 그냥 push하고자 합니다.

![img](https://cdn-images-1.medium.com/max/1600/0*wefv6my2WWLW2385.png)

When do I want `dep.notify()` to be called on `price`’s subscribers? I want them to be called when `price` is set. By the end of the article I want to be able to go into the console and do:

언제 `dep.notify()`가 `price`의 가입자에게 호출될까요? 나는 `price`가 정해지면 그 사람들이 부름 받기를 원한다. 이 아티클의 끝 부분에서 저는 콘솔에 들어가서 다음과 같이 할 수 있게 하고자 합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*1XiVHkIOvqzIVOxNxYlsDA.png)

We need some way to hook into a data property (like `price` or `quantity`) so when it’s accessed we can save the `target` into our subscriber array, and when it’s changed run the functions stored our subscriber array.

우리는 데이터 프로퍼티 (`price` 나 `quantity`와 같은)에 접근 할 방법이 필요합니다. 그래서 접근 할 때 `target`을 우리의 구독자(subscriber) 배열에 저장할 수 있습니다. 그리고 변경되면 subscriber 배열을 저장한 함수를 실행합니다.

### ✅ Solution: Object.defineProperty()

We need to learn about the [Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) function which is plain ES5 JavaScript. It allows us to define getter and setter functions for a property. Lemme show you the very basic usage, before I show you how we’re going to use it with our Dep class.

### ✅ 솔루션 : Object.defineProperty ()

표준 ES5 JavaScript 인 [Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 함수에 대해 알아야합니다. 이는 속성에 대한 getter와 setter 함수를 정의 할 수 있게 해줍니다. Dep 클래스에서 사용하는 방법을 보여드리기 전에 먼저 아주 기본적인 사용법을 보여 드리겠습니다. 

![img](https://cdn-images-1.medium.com/max/1600/1*KLPITQjsRSoGjOBRc6Y8zA.png)

![img](https://cdn-images-1.medium.com/max/1600/0*i1g7DtASO4z1rOvk.png)

As you can see, it just logs two lines. However, it doesn’t actually `get` or `set`any values, since we over-rode the functionality. Let’s add it back now. `get()`expects to return a value, and `set()` still needs to update a value, so let’s add an `internalValue` variable to store our current `price` value.

보시다시피 두 줄을 기록합니다. 그러나 우리가 오버로드했기 때문에 실제로는 값을 `get` 하거나 `set` 하지 않습니다. 지금 다시 추가합시다. `get()`은 값을 반환 할 것으로 기대하고, `set()`은 여전히 값을 갱신 할 필요가 있으므로, 현재의`price` 값을 저장하기 위해 `internalValue` 변수를 추가합시다.

![img](https://cdn-images-1.medium.com/max/1600/1*ek8RxCQ6pkLgbs-DJWteOg.png)

Now that our get and set are working properly, what do you think will print to the console?

이제는 get과 set이 제대로 작동하므로 콘솔에 어떤 내용이 출력 될까요?

![img](https://cdn-images-1.medium.com/max/1600/0*lwD5BfrrNiiZjhyw.png)

So we have a way to get notified when we get and set values. And with some recursion we can run this for all items in our data array, right?

따라서 우리는 값을 얻고 설정할 때 통보 받을 수 있는 방법을 가지고 있습니다. 그리고 재귀를 통해 데이터 배열의 모든 항목에 대해 이 작업을 수행할 수 있습니다.

FYI, `Object.keys(data)` returns an array of the keys of the object.

참고로, `Object.keys(data)`는 객체의 키 배열을 반환합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*56SYVD46rppSBl6mDzzsMg.png)

Now everything has getters and setters, and we see this on the console.

이제는 모든 getter와 setter가 있으며 콘솔에서 볼 수 있습니다.

![img](https://cdn-images-1.medium.com/max/1600/0*VquzAf2KmijoTfbD.png)

### 🛠 Putting both ideas together

### 🛠 개념(idea) 모으기

![img](https://cdn-images-1.medium.com/max/1600/1*2Nvu9DfFpp__k-HTMLSdYg.png)

When a piece of code like this gets run and **gets** the value of `price`, we want `price` to remember this anonymous function (`target`). That way if `price`gets changed, or is **set** to a new value, it’ll trigger this function to get rerun, since it knows this line is dependent upon it. So you can think of it like this.

위와 같은 코드가 실행되면 `price`의 값을 **얻습니다.**  `price`가 이 익명의 함수 (`target`)를 기억하고자 합니다. 이런식으로 `price`가 변경되거나 **set**이 새로운 값으로 설정되면 이 함수는 이 함수가 종속되어 있다는 것을 알고 있기 때문에 이 함수가 다시 실행되도록 트리거합니다. 그래서 이렇게 생각할 수 있습니다.

**Get** => Remember this anonymous function, we’ll run it again when our value changes.

**Set** => Run the saved anonymous function, our value just changed.

Or in the case of our Dep Class

**Price accessed (get)** => call `dep.depend()` to save the current `target`

**Price set** => call `dep.notify()` on price, re-running all the `targets`

Let’s combine these two ideas, and walk through our final code.



**Get** => 익명 함수를 기억한다. 값이 변경되면 다시 실행한다.

**Set** => 저장된 익명 함수를 실행하면 값이 변경된다.

Dep 클래스의 경우

**Price에 접근시 (get)** => `dep.depend()`를 호출하여 현재 `target`을 저장한다.

**Price 설정** => `dep.notify()`를 price에 호출하고 모든 target을 다시 실행한다.

이 두 가지 개념(idea)를 결합하여 최종 코드를 살펴 보겠습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*bM-LGqWYYU3lCaazJ7cAew.png)

And now look at what happens in our console when we play around.

앱을 가지고 놀 때 콘솔에서 어떤 일이 일어나는지 보세요.

![img](https://cdn-images-1.medium.com/max/1600/0*mgmRTNK_n0i2AFK2.png)

Exactly what we were hoping for! Both `price` and `quantity` are indeed reactive! Our total code gets re-run whenever the value of `price` or `quantity`gets updated.

This illustration from the Vue docs should start to make sense now.

우리가 원하는 바가 확실히 이뤄지네요! `price` 와 `quantity`는 모두 반응형입니다! 모든 코드는 `price` 또는 `quantity` 값이 업데이트 될 때마다 재실행됩니다.

Vue 문서의 그림이 이제 이해가 되실겁니다.

![img](https://cdn-images-1.medium.com/max/1600/0*tB3MJCzh_cB6i3mS.png)

Do you see that beautiful purple Data circle with the getters and setters? It should look familiar! Every component instance has a `watcher` instance (in blue) which collects dependencies from the getters (red line). When a setter is called later, it **notifies** the watcher which causes the component to re-render. Here’s the image again with some of my own annotations.

getter와 setter가 들어있는 예쁜 보라색 데이터 동그라미가 보이시나요? 익숙해 보이셔야 합니다! 모든 컴포넌트 인스턴스에는 getter (빨간색 선)의 종속성을 수집하는 `watcher` 인스턴스(파란색)가 있습니다. 나중에 setter가 호출되면 컴포넌트가 다시 렌더링 되도록 하는 감시자에게 **알림**을 보냅니다. 아래는 제 설명이 들어간 이미지입니다.

![img](https://cdn-images-1.medium.com/max/1600/0*P268NBNs64Z-CERj.png)

Yeah, doesn’t this make a whole lot more sense now?

Obviously how Vue does this under the covers is more complex, but you now know the basics.

이제 좀 더 명확해졌죠?

Vue가 커버 아래에서 이 작업을 하는 방법은 분명 더 복잡하지만, 이제는 여러분은 기본적인 것을 알고 있습니다.

### ⏪ So what have we learned?

- How to create a **Dep class** which collects a dependencies (depend) and re-runs all dependencies (notify).
- How to create a **watcher** to manage the code we’re running, that may need to be added (target) as a dependency.
- How to use **Object.defineProperty()** to create getters and setters.

### ⏪ 우리는 무엇을 배웠는가?

- 종속성(dependencies)을 수집하고 (depend) 모든 종속성을 다시 실행하는 (notify) **Dep 클래스**를 작성하는 방법.
- 우리가 실행중인 코드를 관리하기위한 **감시자(watcher)**를 만드는 방법. 종속성으로 추가 (target) 해야 할 수도 있습니다.
- getter 및 setter를 만들기 위해 **Object.defineProperty()**를 사용하는 방법.

### What Next?

If you enjoyed learning with me on this article, the next step in your learning path is to learn about [Reactivity with Proxies](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/). Definitely check out my [my free video](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/) on this topic on VueMastery.com where I also speak with Evan You, the creator of Vue.js.

### 다음은?

이 아티클이 유익하셨다면, 학습 경로의 다음 단계는 [프록시와 반응형](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies)에 대해 배우는 것입니다. VueMastery.com에서 이 주제에 대한 내 [내 무료 비디오](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/)를 꼭 확인하십시오. 여기서 나는 Evan You, Vue.js의 창시자와 해당 주제에 대해 이야기합니다.

------

*Originally published at* *www.vuemastery.com*

