# Evan You on Proxies

> 이 글은 원작자의 허락을 받아 번역하였으며, 글의 출처는 https://www.vuemastery.com/courses/advanced-components/evan-you-on-proxies/ 입니다.
>
> 기술이나 번역에 대해 의견이 있으시면 댓글을 남겨주시면 반영하도록 하겠습니다. 감사합니다.

이전 [강의](http://devtimothy.tistory.com/87)에서 Vue.js의 반응형을 모방한 반응형 시스템을 구축했습니다. 프로퍼티를 `getters / setters`로 변환하기 위해 `Object.defineProperty()`를 사용하면 액세스 할 때 의존성을 추적하고 수정할 때 코드 (알림)를 재실행 할 수 있게 되었습니다.

Vue [로드맵](https://github.com/vuejs/roadmap/blob/master/README.md)를 따르고 있다면 [2.x-next](https://github.com/vuejs /roadmap/blob/master/README.md#2x-next) 버전의 반응형 시스템은 보여드린 것과 다른 프록시로 다시 작성됩니다.

## Core

### 2.6

- 오류 처리, 기능 구성 요소, SSR에 관한 다양한 개선

### 2.x-next

- 네이티브 ES2015 기능을 활용하기 위해서 상시 업데이트 되는 브라우저(evergreen browsers)를 타겟팅합니다
- 반응형 시스템은 여러 가지 개선 된 프록시로 다시 작성됩니다.
- 주요 변경 사항이 없습니다. 기능 패리티가 있는 2.x와 병행하여 유지 관리됩니다.

Evan에게 이것이 정확히 어떻게 생겼는지와 우리가 얻을 수 있는 이점에 대해 물어보고 싶었습니다.

## 장점은 무엇입니까?

[proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) API는 객체의 가상 표현을 생성하고 `set()`,`get()`,`deleteProperty()`  등과 같은 핸들러를 제공합니다.  이는 원래 객체에서 프로퍼티에 액세스하거나 수정할 때 가로 채기 위해 사용할 수 있습니다. 이렇게하면 다음과 같은 제약이 완화됩니다.

- 새로운 반응형 프로퍼티를 추가하기 위한 `Vue.$set()`과 기존 프로퍼티를 지우기 위한 `Vue.$delete()`의 사용.
- [배열 변경 감지](https://vuejs.org/v2/guide/list.html#Caveats).

## 이전 코드

이전에 우리는`Object.defineProperty()`를 사용하여 우리의 프로퍼티가 얻어지고 설정 될 때 리스닝 할 수 있었습니다. 지난 강의에서 마친 위치를 보여주는 [codepen](https://codepen.io/GreggPollack/pen/xjoEOo)이 있습니다.

```javascript
    let data = { price: 5, quantity: 2 };
    let target = null;
    
    // 간단한 Dep 클래스
    class Dep {
      constructor() {
        this.subscribers = [];
      }
      depend() {
        if (target && !this.subscribers.includes(target)) {
          // 타겟이 있고 아직 구독하지 않은 경우에만 push함
          this.subscribers.push(target);
        }
      }
      notify() {
        this.subscribers.forEach(sub => sub());
      }
    }
    
    // 각 데이터 프로퍼티를 살펴봄
    Object.keys(data).forEach(key => {
      let internalValue = data[key];
    
      // 각 프로퍼티는 종속성 인스턴스를 가져옴.
      const dep = new Dep();
    
      Object.defineProperty(data, key, {
        get() {
          dep.depend(); // <-- 실행중인 타겟을 기억함
          return internalValue;
        },
        set(newVal) {
          internalValue = newVal;
          dep.notify(); // <-- 저장된 함수를 재실행
        }
      });
    });
    
    // 반응형 프로퍼티를 수신 대기하는 코드
    function watcher(myFunc) {
      target = myFunc;
      target();
      target = null;
    }
    
    watcher(() => {
      data.total = data.price * data.quantity;
    });
    
    console.log("total = " + data.total)
    data.price = 20
    console.log("total = " + data.total)
    data.quantity = 10
    console.log("total = " + data.total)
```

## 해결 방법 : 프록시를 사용하여 한계를 극복하십시오.

`getters / setters`를 추가하기 위해 각 프로퍼티를 반복하는 대신에 다음을 사용하여 우리의 `data` 객체에 프록시를 설정할 수 있습니다 :

```javascript
    //데이터는 소스 객체
    const observedData = new Proxy(data, { 
      get() {
        //소스 데이터 객체의 프로퍼티에 액세스 할 때 호출
      },
      set() {
        //소스 데이터 객체의 속성이 수정 될 때 호출
      },
      deleteProperty() {
        //소스 데이터 객체의 속성이 삭제 될 때 호출
      }
    });
```

Proxy 생성자 함수에 전달 된 두 번째 인수는 [handler](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler)라고합니다. 핸들러는 트랩이라는 기능을 포함하는 객체일 뿐입니다. 이 트랩은 소스 `data` 오브젝트에서 발생하는 연산을 가로챕니다.

`get()`과 `set()`은 `dep.depend()`와 `dep.notify()`를 각각 호출하는 데 사용할 수 있는 두 개의 트랩입니다.  `set()`트랩은 새롭게 추가 된 프로퍼티에 대해서조차도 호출 될 것이므로 새로운 프로퍼티를 반응형으로 만드는데 사용될 수 있습니다. 따라서 더 이상 `Vue.$set()`을 사용하여 새로운 반응 특성을 선언 할 필요가 없습니다. `deleteProperty()`트랩에서 처리 될 수 있는 반응형 프로퍼티의 삭제에도 똑같이 적용됩니다.

## 프록시를 사용하여 반응형 시스템 구현하기

Proxy API가 아직 Vue의 반응형 시스템에 통합되어 있지는 않지만 Proxy를 사용하여 이전 강의의 반응형 시스템을 구현하려고합니다. 우선 우리가 바꾸어 놓을 우리의 `Object.keys(data).forEach` 루프를 사용합니다.이 루프는 각각의 반응형 특성에 대해 새로운 Dep를 생성하는 데 사용됩니다.

```javascript
    let deps = new Map(); // 모든 데이터의 deps를 map에 저장해보자
    Object.keys(data).forEach(key => {
      // 각 프로퍼티는 종속성 인스턴스를 가져옴
      deps.set(key, new Dep());
    });
```

**사이드 노트 :** `Dep` 클래스는 동일하게 유지됩니다. 이제 우리는 `Object.defineProperty`의 사용을 프록시의 사용으로 대체 할 것입니다 :

```javascript
    let data_without_proxy = data; // 이전 데이터 객체를 저장한다.
    data = new Proxy(data_without_proxy, {
      // 중간에 프록시가 있는 데이터 오버라이드
      get(obj, key) {
        deps.get(key).depend(); // <-- 실행중인 타겟 기억
        return obj[key]; // 원본 데이터 호출
      },
      set(obj, key, newVal) {
        obj[key] = newVal; // 원본 데이터를 새 값으로 설정
        deps.get(key).notify(); // <-- 저장된 함수를 다시 실행
        return true;
      }
    });
```

보시다시피, `data` 오브젝트를 덮어 쓸 때 `Proxy` 오브젝트를 가질 때 사용되는 소스 `data` 오브젝트의 사본을 보유하는 변수`data_without_proxy`를 생성합니다. `get()`과 `set()` 트랩은 두 번째 인자인 핸들러 객체에 프로퍼티로 전달됩니다.

**get (obj, key)** 이것은 프로퍼티에 접근 할 때 호출되는  함수입니다. 그것은 원래의 객체, 즉 `data_without_proxy`를 `obj` 및 액세스 된 프로퍼티의 키로서 수신합니다. 특정 프로퍼티과 관련된 특정 `Dep` 클래스의 `depend()`메소드를 호출합니다. 마침내 그 키와 관련된 값은 `return obj[key]`를 사용하여 리턴됩니다.

**set (obj, key, newVal)** 처음 두 인수는 위에서 언급 한 `get()` 트랩과 같습니다. 세 번째 인수는 새로운 수정 값입니다. 그런 다음 새로운 값을 `obj[key] = newVal`을 사용하여 수정한 프로퍼티로 설정하고 `notify()`메서드를 호출합니다.

## 전체 이동 및 테스트

코드를 조금 변경해야합니다. `total`은 반응형이 될 필요가 없으므로 자체 변수로 추출합니다.

```javascript
    let total = 0;
    watcher(() => {
      total = data.price * data.quantity;
    });
    console.log("total = " + total);
    data.price = 20;
    console.log("total = " + total);
    data.quantity = 10;
    console.log("total = " + total);
```

이제 프로그램을 다시 실행하면 콘솔에 다음 출력이 표시됩니다.

```javascript
    total = 10
    total = 40
    total = 200
```

좋네요. `total`은 `price`와 `quantity`를 업데이트 할 때 갱신됩니다.

## 반응형 프로퍼티 추가하기

이제 프로퍼티를  `data`에 추가 할 수 있어야 합니다. 이것이 `getters/setters`에 대한 프록시를 고려한 이유 중 하나였습니다.  자, 이제 해봅시다.

다음 코드를 추가합니다.

```javascript
    deps.set("discount", new Dep());  // 프로퍼티를 위한 새로운 dep가 필요
    data["discount"] = 5; // 새로운 프로퍼티 추가
    
    let salePrice = 0; 
    
    watcher(() => {  // 반응형 속성을 포함하는 새로운 코드를 watch
      salePrice = data.price - data.discount;
    });
    
    console.log("salePrice = " + salePrice);
    data.discount = 7.5;  // 이는 반응형이 되어야 하고, watcher를 다시 실행함.
    console.log("salePrice = " + salePrice);
```

프로그램이 실행되면 다음과 같은 결과를 볼 수 있습니다.

```javascript
    ....
    salePrice = 15
    salePrice = 12.5
```

`data.discount`가 수정되면 `salePrice`도 업데이트 된다는 것을 알 수 있습니다. 만세! 완성 된 코드는 [여기](https://codepen.io/GreggPollack/pen/gKogaE)에서 볼 수 있습니다

## ReVue

이 강의에서 Evan은 Vue의 향후 버전 (v2.6-next)이 프록시를 사용하여 반응형을 구현할 수 있는 방법에 관해 우리에게 이야기했습니다. 우리는 아래에 대해 더 많이 배웠습니다.

- 현재의 반응형 시스템의 한계
- 프록시가 작동하는 방법
- 프록시를 사용하여 반응형 시스템을 구축하는 방법

다음 비디오에서는 Vue 소스 코드를 살펴보고 반응형이 어디에 있는지 알아 봅니다.