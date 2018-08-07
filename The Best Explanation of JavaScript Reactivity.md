# The Best Explanation of JavaScript Reactivity 🎆

>이 글은 원작자의 허락을 받아 번역하였으며, 글의 출처는 https://medium.com/vue-mastery/the-best-explanation-of-javascript-reactivity-fea6112dd80d 입니다.
>
>기술이나 번역에 대해 의견이 있으시면 댓글을 남겨주시면 반영하도록 하겠습니다. 감사합니다.



많은 프론트엔드 자바스크립트 프레임워크 (예 : Angular, React, and Vue)에는 자체 반응형 엔진이 있습니다. 반응형의 작동 원리를 이해하면 개발 기술을 향상시킬 수 있고 JavaScript 프레임워크를보다 효과적으로 사용 할 수 있습니다. 아래의 영상과 아티클에서는 Vue 소스 코드에서 볼 수 있는 것과 동일한 종류의 반응형(Reactivity)을 빌드합니다.

*기사를 읽는 대신이 비디오를 보는 경우* [*이 시리즈 영상*](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/) 을 보세요. *Vue의 창시자 인 Evan You와 반응형 및 프록시를 논의합니다.*

### 💡 반응형 시스템

Vue의 반응형 시스템은 처음 작동시킬 때 마술처럼 보일 수 있습니다. 이 Vue 앱을 살펴봅시다.

![img](https://cdn-images-1.medium.com/max/1600/1*aLjr0oQBzX7PoUF6oL7yfQ.png)

![img](https://cdn-images-1.medium.com/max/1600/1*neR2Y-0zJseWT8oY1JukXA.png)

Vue는 `price` 값이 바뀌면 세 가지 작업을 하게 됩니다.

- 웹 페이지에서 `price`값을 업데이트
- `price` `*` `quantity`를 곱하는 표현식을 다시 계산하고 페이지를 업데이트
- `totalPriceWithTax` 함수를 다시 호출하고 페이지를 업데이트

여기서 궁금증이 생기실텐데요, `price`가 바뀌면 Vue는 무엇을 업데이트 해야 하는지를 어떻게 알 수 있으며, 어떻게 전부 추적 할 수 있을까요?

![img](https://cdn-images-1.medium.com/max/1600/1*t8enMn6h0gjY6HNKoSVC1g.jpeg)

**이는 자바스크립트 프로그래밍이 정상적으로 작동하는 방식이 아닙니다**

이 말이 명확하게 느껴지지 않으시다면, 우리가 다룰 쟁점은 프로그래밍이 대개 이런 식으로 작동하지 않는다는 점입니다. 예를 들어서 아래 코드를 실행하면 다음과 같습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*RrDCv_fUYnOl34Eq0afgYw.png)

어떤 값이 나올거 같나요? Vue를 사용하는것이 아니므로 `10`을 나타낼 겁니다.

![img](https://cdn-images-1.medium.com/max/1600/1*q7nHV9seYErboH1DDiEdUg.png)

Vue에서는 `total` 또는 `quantity`가 업데이트 될 때마다 `total`이 업데이트 되기를 바랍니다.

![img](https://cdn-images-1.medium.com/max/1600/1*aFGF-Go7ONnOtWjfyzytig.png)

안타깝게도 JavaScript는 절차형이지, 반응형이 아니므로 실제로는 동작하지 않습니다. `total`을 반응형으로 만들기 위해서 JavaScript를 사용하여 다르게 작동하게 합니다.

### ⚠️ 문제

`total`을 계산해야하기 때문에 `price` 또는`quantity`가 변경 될 때마다 재실행해야 한다.

### ✅ 솔루션

우선적으로는 앱에 알릴 수 있는 방법이 필요합니다. "내가 실행하려고 하는 코드를 **저장해 두었다가** 다른 때 실행할 수도 있습니다." 그러면 원할 때 코드를 실행 할 수 있겠죠. `price` 또는 `quantity` 변수가 업데이트 되면, 저장된 코드를 다시 실행하게 합니다.

![img](https://cdn-images-1.medium.com/max/1600/0*Nh-FQoHiDHncmQSi.png)

함수를 다시 실행하기 위해서 이 함수를 기록해 둡니다.

![img](https://cdn-images-1.medium.com/max/1600/1*vD19ImKAK2WYySJGIvrKtQ.png)

익명의 함수를 `target` 변수 안에 저장하고,`record` 함수를 호출합니다. ES6 화살표 구문을 사용하여 다음과 같이 작성할 수도 있습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*z0E_bw1_XBjGdM0ab_pyDg.png)

`record`의 정의는 간단합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*6tRbHmwr7mzy5CNemhkTcg.png)

우리는 `target`을 저장하고 있습니다. (해당 코드의 경우 `{ total = price * quantity }`). 기록한 모든 것을 실행하는 `replay` 함수를 나중에 실행할 수도 있습니다. .

![img](https://cdn-images-1.medium.com/max/1600/1*dyxkUSFl1S3m4dFCgc2jZw.png)

이것은 스토리지 배열 내부에 저장된 모든 익명 함수을 거쳐 각각을 실행합니다.

그러면 코드에서 다음과 같이 할 수 있습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*Fr8Oif-PkvmyFDKDt5Xm6w.png)

간단하죠? 아래는 전체 코드입니다. 처음부터 끝까지 쭉 읽고 이해하고자 하는 경우 참고하세요. 

![img](https://cdn-images-1.medium.com/max/1600/1*TpEBEstjfM4FNMYuh37BLA.png)

![img](https://cdn-images-1.medium.com/max/1600/0*0a_165xKF15xL889.png)

### ⚠️ 문제

필요에 따라서 기록할 target을 실행할 수도 있지만, 앱으로 확장할 수 있는 더 강력한 솔루션을 갖는게 좋을 것입니다. 아마도 재실행을 원할 때, 알림을 받는 타겟 목록을 유지 관리하는 클래스일 것입니다.

### ✅ 솔루션 : 종속 클래스

이 문제를 해결하기 위해 시작할 수 있는 한 가지 방법은 자체 클래스에 이 동작을 캡슐화 함으로써, 관찰자 패턴(observer pattern)을 **종속 클래스**로 구현하는 것입니다.

따라서 종속성 (Vue가 일을 처리하는 방법에 더 가깝습니다.)을 관리하기 위한 JavaScript 클래스를 만든다면 다음과 같을 것입니다.

![img](https://cdn-images-1.medium.com/max/1600/1*9NnQmGxZfmxhRJBUEs4Z7g.png)

`storage` 대신에 익명 함수를 `subscribers`에 저장하고 있습니다. `record` 함수 대신에 `depend`을 호출하고 `replay` 대신 `notify`를 사용합니다. 이를 실행하려면 다음 코드를 작성하세요.

![img](https://cdn-images-1.medium.com/max/1600/1*Y5XJpipq7-Po1mP_eJoCGw.png)

여전히 작동하며 코드가 더 재사용 가능해 졌습니다. 조금 이상한 건  `target` 설정과 실행뿐입니다.

### ⚠️ 문제

앞으로는 각 변수에 대해 Dep 클래스를 갖게 될 것이며, 업데이트가 필요한 익명 함수를 만드는 동작을 캡슐화하는 것이 좋습니다. 아마도 `감시자 (watcher)` 기능이 이러한 행동을 처리할 겁니다.

![img](https://cdn-images-1.medium.com/max/1600/1*mo9tPOcAy-qC1VZ6Bnz76A.png)

(위에서 작성한 코드와 같습니다.)

위 코드를 호출하는 대신에, 다음을 호출하도록 합시다.

![img](https://cdn-images-1.medium.com/max/1600/1*2TPYfKaV4UBEReN8PWwzbQ.png)

### ✅ 솔루션 : 감시자 (Watcher) 함수

Watcher 함수에서 우리는 몇 가지 간단한 일을 할 수 있습니다 :

![img](https://cdn-images-1.medium.com/max/1600/1*U7bJcE5Ad7lxbQUP-U68uw.png)

보시다시피, `watcher` 함수는 `myFunc` 인수로 취해서  전역 `target` 프로퍼티로 설정하고 `dep.depend()`를 호출하여 타겟을 구독자(subscriber)로 추가하고`target` 함수를 호출하고, `target`을 리셋합니다.

아래는 실행 결과입니다.

![img](https://cdn-images-1.medium.com/max/1600/1*vefCUnWyacq0GxC3TRI0Cw.png)

![img](https://cdn-images-1.medium.com/max/1600/0*D05AHN0_GUXoMVM8.png)

You might be wondering why we implemented `target` as a global variable, rather than passing it into our functions where needed. There is a good reason for this, which will become obvious by the end of our article.

`target`을 전역 변수로 구현 한 이유가 궁금하실텐데요, 이는 우리 아티클의 마지막 부분 그 이유가 명확해질 겁니다.

### ⚠️ 문제

우리는 하나의 `Dep class`를 가지고 있지만, 우리가 정말로 원하는 것은 각각의 변수가 자신의 Dep를 갖도록 하는 것입니다. 진도를 더 나가기 전에 프로퍼티를 살펴봅시다.

![img](https://cdn-images-1.medium.com/max/1600/1*YBknbJTkI-za0L9eMAFayQ.png)

우리의 각 프로퍼티 (`price` 와 `quantity`)가 그들 자신의 내부 Dep 클래스를 가지고 있다고 가정해 봅시다.

![img](https://cdn-images-1.medium.com/max/1600/0*kV4iCRoguwO5C_JQ.png)

아래 코드를 실행시킵시다.

![img](https://cdn-images-1.medium.com/max/1600/1*-rznzvwxr5clvYdPVq2MfA.png)

`data.price` 값에 접근했으므로 `price` 프로퍼티의 Dep 클래스가 `target`에 저장되어 있는 익명 함수를 subscriber 배열에 push하고자 합니다(`dep.depend()` 를 호출함으로서). `data.quantity`에 접근했으므로 `amount` 프로퍼티의 Dep 클래스가 이 `target`에 저장되어 있는 익명 함수를 subscriber 배열에 push 하고자 합니다.

![img](https://cdn-images-1.medium.com/max/1600/0*E-_YXfn3vJe7S_Ry.png)

`data.price` 만 액세스하는 또 다른 익명의 함수가 있으면 `price` 속성 Dep 클래스에 그냥 push하고자 합니다.

![img](https://cdn-images-1.medium.com/max/1600/0*wefv6my2WWLW2385.png)

언제 `dep.notify()`가 `price`의 구독자(subscribers)에게 호출되게 할까요? `price`가 정해지면 함수가 호출되게 하고자 합니다. 이 아티클의 끝 부분에서 저는 콘솔에 들어가서 다음과 같이 할 수 있게 하고자 합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*1XiVHkIOvqzIVOxNxYlsDA.png)

우리는 데이터 프로퍼티 (`price` 나 `quantity`와 같은)에 접근할 방법이 필요합니다. 그래서 접근 할 때 `target`을 구독자(subscriber) 배열에 저장할 수 있습니다. 그리고 변경되면 subscriber 배열을 저장한 함수를 실행합니다.

### ✅ 솔루션 : Object.defineProperty()

표준 ES5 JavaScript 인 [Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 함수에 대해 알아야합니다. 이는 프로퍼티에 대한 getter와 setter 함수를 정의 할 수 있게 해줍니다. Dep 클래스에서 사용하는 방법을 보여드리기 전에 먼저 아주 기본적인 사용법을 보여 드리겠습니다. 

![img](https://cdn-images-1.medium.com/max/1600/1*KLPITQjsRSoGjOBRc6Y8zA.png)

![img](https://cdn-images-1.medium.com/max/1600/0*i1g7DtASO4z1rOvk.png)

보시다시피 두 줄의 로그가 찍힙니다. 그러나 우리가 오버로드했기 때문에 실제로는 값을 `get` 하거나 `set` 하지 않습니다. 다시 코드를 추가합시다. `get()`은 값을 반환 할 것으로 기대하고, `set()`은 여전히 값을 갱신 할 필요가 있으므로, 현재의`price` 값을 저장하기 위해 `internalValue` 변수를 추가합시다.

![img](https://cdn-images-1.medium.com/max/1600/1*ek8RxCQ6pkLgbs-DJWteOg.png)

이제 get과 set이 제대로 작동하므로 콘솔에 어떤 내용이 출력 될까요?

![img](https://cdn-images-1.medium.com/max/1600/0*lwD5BfrrNiiZjhyw.png)

따라서 우리는 값을 얻고 설정할 때 통보 받을 수 있는 방법을 가지고 있습니다. 그리고 재귀를 통해 데이터 배열의 모든 항목에 대해 이 작업을 수행할 수 있습니다.

참고로, `Object.keys(data)`는 객체의 키 배열을 반환합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*56SYVD46rppSBl6mDzzsMg.png)

이제는 모든 getter와 setter가 있으며 콘솔에서 볼 수 있습니다.

![img](https://cdn-images-1.medium.com/max/1600/0*VquzAf2KmijoTfbD.png)

### 🛠 개념(idea) 모으기

![img](https://cdn-images-1.medium.com/max/1600/1*2Nvu9DfFpp__k-HTMLSdYg.png)

위와 같은 코드가 실행되면 `price`의 값을 **얻습니다.**  `price`가 이 익명의 함수 (`target`)를 기억하고자 합니다. 이런식으로 `price`가 변경되거나 **set**이 새로운 값으로 설정되면 이 함수는 이 함수가 종속되어 있다는 것을 알고 있기 때문에 이 함수가 다시 실행되도록 트리거합니다. 그래서 이렇게 생각할 수 있습니다.

**Get** => 익명 함수를 기억한다. 값이 변경되면 다시 실행한다.

**Set** => 저장된 익명 함수를 실행하면 값이 변경된다.

Dep 클래스의 경우

**Price에 접근시 (get)** => `dep.depend()`를 호출하여 현재 `target`을 저장한다.

**Price 설정** => `dep.notify()`를 price에 호출하고 모든 target을 다시 실행한다.

이 두 가지 개념(idea)를 결합하여 최종 코드를 살펴 보겠습니다.

![img](https://cdn-images-1.medium.com/max/1600/1*bM-LGqWYYU3lCaazJ7cAew.png)

앱을 가지고 놀 때 콘솔에서 어떤 일이 일어나는지 보세요.

![img](https://cdn-images-1.medium.com/max/1600/0*mgmRTNK_n0i2AFK2.png)

우리가 원하는 바가 확실히 이뤄지네요! `price` 와 `quantity`는 모두 반응형입니다! 모든 코드는 `price` 또는 `quantity` 값이 업데이트 될 때마다 재실행됩니다.

Vue 문서의 그림이 이제 이해가 되실겁니다.

![img](https://cdn-images-1.medium.com/max/1600/0*tB3MJCzh_cB6i3mS.png)

getter와 setter가 들어있는 예쁜 보라색 데이터 동그라미가 보이시나요? 친숙해 보이셔야 합니다! 모든 컴포넌트 인스턴스에는 getter (빨간색 선)의 종속성을 수집하는 `watcher` 인스턴스(파란색)가 있습니다. 나중에 setter가 호출되면 컴포넌트가 다시 렌더링 되도록 하는 감시자에게 **알림**을 보냅니다. 아래는 제 설명이 들어간 이미지입니다.

![img](https://cdn-images-1.medium.com/max/1600/0*P268NBNs64Z-CERj.png)

야호, 이제 좀 더 명확해졌죠?

Vue가 커버 아래에서 이 작업을 하는 방법은 분명 더 복잡하지만, 이제는 여러분은 기본적인 것을 알고 있습니다.

### ⏪ 우리는 무엇을 배웠는가?

- 종속성(dependencies)을 수집하고 (depend) 모든 종속성을 다시 실행하는 (notify) **Dep 클래스**를 작성하는 방법.
- 우리가 실행중인 코드를 관리하기위한 **감시자(watcher)**를 만드는 방법. 종속성으로 추가 (target) 해야 할 수도 있습니다.
- getter 및 setter를 만들기 위해 **Object.defineProperty()**를 사용하는 방법.

### 다음은?

이 아티클이 유익하셨다면, 학습 경로의 다음 단계는 [프록시와 반응형](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies)에 대해 배우는 것입니다. VueMastery.com에서 이 주제에 대한 [내 무료 비디오](https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/)를 꼭 확인하십시오. 여기서 나는 Vue.js의 창시자 Evan You와 해당 주제에 대해 이야기합니다.

------

*Originally published at* *www.vuemastery.com*