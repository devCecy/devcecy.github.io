---
layout: post
title: "[함수형 프로그래밍] 이터러블(Iterable) 직접 만들어보기"
date: 2022-07-24 21:54:36 +0530
categories: 함수형프로그래밍
---

오늘은 앞서 배운 내용을 기반으로 사용자 정의 이터러블을 만들어 보겠습니다.

</br>

# 이터러블/이터레이터 정의대로 뼈대 만들기

이터러블을 만들기 위해 이터러블과 이터레이터의 정의를 떠올려 보려고 합니다.
이터러블은 이터레이터를 리턴하며, [Symbol.iterator]()를 가진 값이고, 이터레이터는 {value, done} 객체를 리턴하며, next()를 가진 값입니다. 이 정의대로 이터러블을 만들어봅시다.

```javascript
const iterable = {
	// 이터러블은
	[Symbol.iterator]() {
		// 심볼 이터레이터 메서드를 구현하고 있어야합니다.
		return {
			// 심볼 이터레이터는 이터레이트를 반환해야합니다.
			next() {
				// 이터레이터는 next를 메서드로 가지고 있으며,
				return { value, done }; // next메서드는 value와 done을 가진 객체를 리턴합니다.
			},
		};
	},
};
```

우리의 이터러블이 그럴듯한 모습을 갖추게되었습니다. 이제 우리의 이터러블을 3, 2, 1이라는 value값을 리턴해주는 이터레이터로 만들어봅시다. i는 3이고, next메서드가 호출되면 i--된 value값을 보여주려고 합니다. 우리가 만든 이터러블이 리턴한 이터레이터를 next메서드로 호출해보니 기대한대로 3, 2, 1 value값이 리턴되고있습니다.

```javascript
const iterable = {
	[Symbol.iterator]() {
		let i = 3;
		return {
			next() {
				// i가 3,2,1이면 i값을 하나씩 감소시키며 done은 false를 반환하고, i가 0이면 done은 true가 됩니다.
				return i === 0 ? { done: true } : { value: i--, done: false };
			},
		};
	},
};

let iterator = iterable[Symbol.iterator](); // 위와같은 iterable은 심볼이터레이터를 통해 interator를 리턴할 수 있습니다.
// iterator는 next메서드를 통해 내부의 값을 조회할 수 있습니다.
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // { done: false}
```

또한, iterable에 [Symbol.iterator]가 구현되어있기 때문에 for-of문을 돌릴 수 있습니다. 다시말해, 리턴 된 iterator가 value값으로 3, 2, 1을 반환해 주기 대문에 for-of문의 iterable부분에 3, 2, 1이 들어가 반복문을 순회할 수 있게 됩니다.

```javascript
const iterable = {
	[Symbol.iterator]() {
		let i = 3;
		return {
			next() {
				return i === 0 ? { done: true } : { value: i--, done: false };
			},
		};
	},
};

for (const a of iterable) console.log(a); // 3, 2, 1
```

</br>

# well-formed 이터러블 만들기

여기까지 순회하는 이터러블의 모습을 만들어보았습니다. 그러나 아직은 잘 만들어진(well-formed) 이터러블이라고 하기에는 조금 부족합니다. 무엇이 문제인지 확인해보기 위해서 iterable이 리턴한 이터레이터를 iterator라는 변수에 담아 for-of문을 돌려봅시다. 앗, 이상합니다. iterator가 이터러블이 아니라고 합니다(?!). 왜그럴까요??

```javascript
const iterable = {
	[Symbol.iterator]() {
		let i = 3;
		return {
			next() {
				return i === 0 ? { done: true } : { value: i--, done: false };
			},
		};
	},
};

let iterator = iterable[Symbol.iterator]();

for (const a of iterator) console.log(a); // iterator is not iterable
```

well-formed 이터러블의 예를 봅시다.

변수 iterator로 next 메서드를 한번 호출하여 {value: 1, done: false} 값을 뽑고난 후, for-of문을 순회하면 이미 반환된 자기 자신의 상태에 이어 다음 값을 반환해줍니다. 또한, iterator와 iterator[Symbol.iterator]()값이 일치(true)하는것보니 iterator가 자기 자신을 반환하는 Symbol.iterator를 가지고 있습니다.

```javascript
const arr = [1, 2, 3];
let iterator = arr[Symbol.iterator]();
iterator.next(); // {value: 1, done: false}
console.log(iterator[Symbol.iterator]() === iterator); // true
for (const a of iterator) console.log(a); // 2, 3
```

그런데 우리가 만든 사용자정의의 iterator는 단지 value, done을 가진 객체만 리턴하며, 자기 자신을 반환하지 않아 for-of문을 순회할 수 없습니다. Symbol.iterator를 실행한 이터레이터가 자기 자신 또한 이터러블이면서, Symbol.iterator를 실행했을 때 자기 자신을 반환하도록 해야 well-formed 이터러블이라고 할 수 있습니다.

```javascript
const iterable = {
	[Symbol.iterator]() {
		let i = 3;
		return {
			next() {
				return i === 0 ? { done: true } : { value: i--, done: false };
			},
			[Symbol.iterator]() {
				return this;
			}, // 자기 자신을 반환하는 값을 가지도록 합니다.
		};
	},
};
```

정리해보면, well-formed 이터러블은

- 이터러블을 for-of문에 넣어 순회가 가능하고,
- 이터러블을 이터레이터로 만든 상태에서 for-of문을 순회 가능하고,
- next메서드로 일정부분 이터레이터를 진행한 후에 for-of문을 실행해도 순회가 됩니다.

</br>

# 이터러블/이터레이터 프로토콜을 따르는 문법

이터러블/이터레이터 프로토콜은 es6의 문법에서만 구현되어있는게 아닙니다. 오픈소스나, 자바스크립트에서 순회가 가능한 값을 가진다면 대부분 이터러블/이터레이터 프로토콜을 따르고 있습니다.

예를들어, document.querySelectorAll을 for-of문으로 돌려보면 아래와같이 순회가 가능합니다. 그리고 Symbol.iterator가 구현되어있어 next메서드로 내장 값을 반환 받을 수도 있습니다.

![함수형프로그래밍](/assets/images/functional-programming/functional-programming-3-1.png)

또한, 전개연산자(...)도 이터러블/이터레이터 프로토콜을 따릅니다.

아래 예시를 보면 전개연산자를 통해 a를 순회해서 1과 2를 뽑아내고, [3, 4]를 순회해서 3과 4를 뽑아냈습니다.

```javascript
const a = [1, 2];
console.log([...a, ...[3, 4]]); //[1, 2, 3, 4]
```

그러나 a[Symbol.iterator]에 null값을 할당한다면 어떻게 될까요? 역시나, iterater가 아니라 순회할 수 없다고 하네요.

```javascript
const a = [1, 2];
a[Symbol.iterator] = null;
console.log([...a, ...[3, 4]]); // a is not iterator
```

</br>

# 마무리

오늘은 직접 이터러블을 구현해 보면서 well-formed 이터러블은 어떤것인지 알아보았습니다. 직접 이터러블을 만들어 보고나니 아주 조금 더 이터러블과 가까워진 기분이 듭니다. 다음시간에는 제너레이터에 대해서 알아보겠습니다😎.

</br>

# 참고

- 유인동 님의 "함수형 프로그래밍과 Javascript ES6+" 강의를 바탕으로 작성했습니다.

- 강의를 들으며 저의 해석을 덧붙였습니다.
