---
layout: post
permalink: /exploring-the-angular-1-5-component-method
title: Exploring the Angular 1.5 .component() method
path: 2015-11-13-exploring-the-angular-1-5-component-method.md
original: http://toddmotto.com/exploring-the-angular-1-5-component-method/
---

Angular 1.5에서는 `component()`라는 헬퍼 메소드를 소개하고 있다. `directive()` 메소드를 사용한 정의에 비해 훨씬 간단하게 일반적인 기본 동작과 모범적인 예를 활용할 수 있게 지원한다. `.component()`를 사용하는 것으로 Angular 2의 스타일을 사용해 작성할 수 있으며 차후 버전에도 적합하다.

`.directive()`메소드와 `.component()` 메소드에서 사용하는 문법을 비교해보고 `component()`가 제공하는 멋진 추상을 살펴보자.


_노트: Angular 1.5는 여전히 `beta`므로 이 버전의 출시를 눈여겨 보자._

### .directive() 에서 .component() 로

이 문법 변경은 아주 간단하다:

{% highlight javascript %}
// before
module.directive(name, fn);

// after
module.component(name, options);
{% endhighlight %}

`name` 인자는 컴포넌트로 정의하고 싶은 이름이며, `options` 인자에는 함수를 넣었던 1.4 그 이하 버전에 문법과 달리 이 컴포넌트에 전달하고자 하는 객체 정의가 들어간다.

간단한 `counter` 컴포넌트를 미리 만들었는데 Angular `1.4.x`에서 사용한 문법을 `.component()` 메소드를 활용해 `v1.5.0`에 맞게 리팩토링 하는 과정을 보여주려고 한다.

{% highlight javascript %}
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    },
    controller: function () {
      function increment() {
        this.count++;
      }
      function decrement() {
        this.count--;
      }
      this.increment = increment;
      this.decrement = decrement;
    },
    controllerAs: 'counter',
    template: [
      '<div class="todo">',
        '<input type="text" ng-model="counter.count">',
        '<button type="button" ng-click="counter.decrement();">-</button>',
        '<button type="button" ng-click="counter.increment();">+</button>',
      '</div>'
    ].join('')
  };
});
{% endhighlight %}

`1.4.x` 디렉티브의 라이브 코드는 [여기서 확인](http://jsfiddle.net/toddmotto/avdezer7/embedded/result,js,html)할 수 있다.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/avdezer7/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Angular 1.4 버전에서 만든 이 버전을 변경하며 그 변화를 살펴보기로 한다.

### 함수를 객체로, 메소드 이름의 변화

`function` 인자를 `Object`로 변경하는 것을 먼저 해보자. 그리고서 이름을 `.directive()`에서 `.component()`로 변경한다:

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    
  };
});

// after
.component('counter', {
    
});
{% endhighlight %}

간단하고 좋다. `.directive()`에서 `return {};`가 필수적이었던 반면 `.component()`의 객체 사용은 훨씬 단순하게 보인다.

### "scope"와 "bindToController"를 "bindings"로 변경

`.directive()` 메소드에서 `scope` 프로퍼티는 `$scope`를 고립할지 혹은 상속할지에 대한 정의를 위해 활용했는데 대부분의 경우 기본적으로 모든 스코프가 고립된 형태로 정의를 했다. 그래서 매번 디렉티브를 만들 때마다 과도하게 스코프를 매번 정의해야 하는 불편함이 있었다. `bindToController`가 소개된 후, 프로퍼티를 스코프에 넘기는지, 또는 컨트롤러에 바로 연결하는지를 명시적으로 선언할 수 있게 되었다.


`.component()`의 `bindings` 프로퍼티는 이런 반복적인 기초 작업을 제거했다. 컴포넌트가 고립된 스코프를 갖는다는 가정을 기본적으로 하고서, 간단하게 컴포넌트에 내려주고 싶은 데이터만 정의해주면 된다.

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    }
  };
});

// after
.component('counter', {
  bindings: {
    count: '='
  }
});
{% endhighlight %}

### Controller와 controllerAs의 변경

`controller`의 정의는 변경된 바가 없지만 좀 더 똑똑해졌다.

컴포넌트에 컨트롤러를 그 자리에서 선언할 때는 이렇게 작성했을 것이다:

{% highlight javascript %}
// 1.4
{
  ...
  controller: function () {}
  ...
}
{% endhighlight %}

컨트롤러를 다른 곳에서 정의했을 때는 이렇게 했을 것이다:

{% highlight javascript %}
// 1.4
{
  ...
  controller: 'SomeCtrl'
  ...
}
{% endhighlight %}

`controllerAs`를 선언하고 싶을 때는, 새로운 프로퍼티를 추가해서 인스턴스의 별칭을 지정해야 했다.

{% highlight javascript %}
// 1.4
{
  ...
  controller: 'SomeCtrl',
  controllerAs: 'something'
  ...
}
{% endhighlight %}

이 과정으로 `template` 내에서 컨트롤러의 인스턴스를 활용할 때 `something.prop`을 사용하는 것이 가능해졌다.

이제 `.component()`으로 변경되면서 `controllerAs` 프로퍼티를 센스있게 추측해서 자동으로 생성한다. 다음 코드에서 볼 수 있는 것처럼 사용 가능성이 있는 다음 세가지 이름을 자동으로 배정해준다:

{% highlight javascript %}
// inside angular.js
controllerAs: identifierForController(options.controller) || options.controllerAs || name,
{% endhighlight %}

`identifierForController` 함수가 컨트롤러의 이름을 추측하는 방법은 다음과 같다:

{% highlight javascript %}
// inside angular.js
var CNTRL_REG = /^(\S+)(\s+as\s+(\w+))?$/;
function identifierForController(controller, ident) {
  if (ident && isString(ident)) return ident;
  if (isString(controller)) {
    var match = CNTRL_REG.exec(controller);
    if (match) return match[3];
  }
}
{% endhighlight %}

이 함수로 `.component()`에서 다음과 같은 문법을 사용할 수 있게 된다:

{% highlight javascript %}
// 1.5
{
  ...
  controller: 'SomeCtrl as something'
  ...
}
{% endhighlight %}

이 기능이 `controllerAs` 프로퍼티를 추가하지 않게 만든다... _하지만_...

`controllerAs` 프로퍼티를 하위 호환성을 위해, 또는 디렉티브/컴포넌트를 작성하는 스타일을 유지하기 위해 계속 사용할 수도 있다.

그렇게 좋은 방법은 아니지만 세번째 선택지로, `controllerAs`를 완벽하게 다 지워버리고 Angular가 자동으로 해당 컴포넌트의 이름을 별칭으로 사용하게 하는 방법을 사용 할 수 있다. 예를 들면:

{% highlight javascript %}
.component('test', {
  controller: function () {
    this.testing = 123;
  }
});
{% endhighlight %}

여기서 `controllerAs`의 정의는 자동으로 `test`가 된다. 그래서 `template`에서 `test.testing`을 사용하면 `123` 값을 반환할 것이다.

이 방법으로 `controller`를 추가하고 앞서 작성한 디렉티브를 컴포넌트로 변경하며 `controllerAs` 프로퍼티를 제거할 수 있다:


{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    },
    controller: function () {
      function increment() {
        this.count++;
      }
      function decrement() {
        this.count--;
      }
      this.increment = increment;
      this.decrement = decrement;
    },
    controllerAs: 'counter'
  };
});

// after
.component('counter', {
  bindings: {
    count: '='
  },
  controller: function () {
    function increment() {
      this.count++;
    }
    function decrement() {
      this.count--;
    }
    this.increment = increment;
    this.decrement = decrement;
  }
});
{% endhighlight %}

변경된 방법으로 정의하고 사용하는 것이 훨씬 간단하다.

### 템플릿

`template`에도 적어둘 만한, 세밀한 변화가 있다. `template` 프로퍼티를 추가하고 어떤지 확인하자.

{% highlight javascript %}
.component('counter', {
  bindings: {
    count: '='
  },
  controller: function () {
    function increment() {
      this.count++;
    }
    function decrement() {
      this.count--;
    }
    this.increment = increment;
    this.decrement = decrement;
  },
  template: [
    '<div class="todo">',
      '<input type="text" ng-model="counter.count">',
      '<button type="button" ng-click="counter.decrement();">-</button>',
      '<button type="button" ng-click="counter.increment();">+</button>',
    '</div>'
  ].join('')
});
{% endhighlight %}

`template` 프로퍼티는 이제 함수로 정의해서 `$element`와 `$attrs`를 주입하는 형태로 사용할 수 있다. 만약 `template` 프로퍼티에 함수를 _넣으면_ 컴파일 할 수 있는 HTML 문자열을 반환해야 한다.

{% highlight javascript %}
{
  ...
  template: function ($element, $attrs) {
    // access to $element and $attrs
    return [
      '<div class="todo">',
        '<input type="text" ng-model="counter.count">',
        '<button type="button" ng-click="counter.decrement();">-</button>',
        '<button type="button" ng-click="counter.increment();">+</button>',
      '</div>'
    ].join('')
  }
  ...
}
{% endhighlight %}

동작하는 예제를 확인하자. Angular 버전 `v1.5.0-build.4376+sha.aff74ec` [예제](http://jsfiddle.net/toddmotto/xqauz9aa/embedded/result,js,html)다:

<iframe width="100%" height="300" src="http://jsfiddle.net/toddmotto/xqauz9aa/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

여기까지 디렉티브를 컴포넌트로 변경하는 과정이었다. 여기서 끝내기 전에 살펴봐야 하는 변경점이 몇가지 더 있다.

### 끼워넣기가 가정되어 있음 Assumed transclusion

컴포넌트는 기본적으로 끼워넣기(Transclusion)를 가정하고 있는데, 다음 Angular 코드에 의해 설정된다:

{% highlight javascript %}
// angular.js
{
  ...
  transclude: options.transclude === undefined ? true : options.transclude
  ...
}
{% endhighlight %}

이 기능을 끄고 싶다면 `transclude: false`로 정의하면 된다.

### 고립된 스코프 끄기

컴포넌트는 스코프 고립이 기본값이다. `.component()`에서 이 설정을 바꾸고 싶다면 간단하게 프로퍼티로 정의하면 된다:

{% highlight javascript %}
.component('counter', {
  isolate: false
});
{% endhighlight %}

다음 Angular의 삼항 연산자에 따라서 자동으로 `scope`에 빈 객체를 넣게 된다. `.directive()`에서 상속하는 방식대로 사용하고 싶다면 고립된 스코프를 끄면 된다. 그러면 `scope: true`로 동작한다. 내부 코드는 다음과 같다:

{% highlight javascript %}
{
  ...
  scope: options.isolate === false ? true : {}
  ...
}
{% endhighlight %}

### 비교를 위한 소스코드

이 글 내내 Angular 소스 코드 스니핏을 교차 레퍼런스로 활용했다. 코드를 보고 싶다면 [여기에서](https://github.com/angular/angular.js/blob/54e816552f20e198e14f849cdb2379fed8570c1a/src/loader.js#L362-L396) 확인하거나 아래 코드를 확인해보면 좋겠다. 정말 좋은 추상화 구현이다:

{% highlight javascript %}
component: function(name, options) {
  function factory($injector) {
    function makeInjectable(fn) {
      if (angular.isFunction(fn)) {
        return function(tElement, tAttrs) {
          return $injector.invoke(fn, this, {$element: tElement, $attrs: tAttrs});
        };
      } else {
        return fn;
      }
    }

    var template = (!options.template && !options.templateUrl ? '' : options.template);
    return {
      controller: options.controller || function() {},
      controllerAs: identifierForController(options.controller) || options.controllerAs || name,
      template: makeInjectable(template),
      templateUrl: makeInjectable(options.templateUrl),
      transclude: options.transclude === undefined ? true : options.transclude,
      scope: options.isolate === false ? true : {},
      bindToController: options.bindings || {},
      restrict: options.restrict || 'E'
    };
  }

  if (options.$canActivate) {
    factory.$canActivate = options.$canActivate;
  }
  if (options.$routeConfig) {
    factory.$routeConfig = options.$routeConfig;
  }
  factory.$inject = ['$injector'];

  return moduleInstance.directive(name, factory);
}
{% endhighlight %}

다시 말하지만 Angular 1.5는 아직 릴리즈되지 않았다. 그래서 이 글에서 사용한 API는 _아마_ 조금은 달라질 수 있다.

### Angular 2 로 업그레이드하기

`.component()`를 사용해서 작성하는 스타일은 추후 Angular 2 로 옮기는데 도움이 된다. Angular 2의 문법에서 컴포넌트는 ECMAScript 5와 새로운 템플릿 문법을 활용하고 있긴 하지만 말이다:
Writing components in this style will allow you to upgrade your Components using `.component()` into Angular 2 very easily, it'd look something like this in ECMAScript 5 and new template syntax:

{% highlight javascript %}
var Counter = ng
.Component({
  selector: 'counter',
  template: [
    '<div class="todo">',
      '<input type="text" [(ng-model)]="count">',
      '<button type="button" (click)="decrement();">-</button>',
      '<button type="button" (click)="increment();">+</button>',
    '</div>'
  ].join('')
})
.Class({
  constructor: function () {
    this.count = 0;
  },
  increment: function () {
    this.count++;
  },
  decrement: function () {
    this.count--;
  }
});
{% endhighlight %}
