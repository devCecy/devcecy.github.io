---
layout: post
title: "[함수형 프로그래밍] 이터러블(Iterable)/이터레이터(Iterator) 뜯어보기 (feat. es6의 리스트순회)"
date: 2022-07-18 21:54:36 +0530
categories: 함수형프로그래밍
---

# es5와 es6의 리스트 순회 비교

이터러블, 이터레이터에 대해 알아보기 전에 es5와 es6에서의 리스트 순회를 비교해 봅시다.

## es5의 리스트 순회

반복을 어떻게 해야할지 for문안에 명령적으로 적어주어야 합니다.

```javascript
const list = [1, 2, 3];
const str = "abc";

for (var i = 0; i < list.length; i++) {
	console.log(list[i]); // 1, 2, 3
}

for (var i = 0; i < str.length; i++) {
	console.log(str[i]); // a, b, c
}
```

## es6의 리스트 순회

es5보다 코드도 간결해지고, 선언적으로 코드가 작성되는 것을 알 수 있습니다.

```javascript
const list = [1, 2, 3];
const str = "abc";

for (const a of list) {
	console.log(a); // 1, 2, 3
}

for (const a of str) {
	console.log(a); // a, b, c
}
```

# Array, Set, Map을 통해 알아보는 이터러블/이터레이터

Array와 Set, Map 모두 for-of문을 돌려보았습니다. 마지막 콘솔로그들을 보면 arr[0]은 1을 출력하는데, set[0]과 map[0]은 undefined를 출력합니다. Set과 Map은 내부가 Array와 같지 않음을 짐작해 볼 수 있습니다.

```javascript
const arr = [1, 2, 3];
for (const a of arr) console.log(a); // 1, 2, 3

const set = new Set([1, 2, 3]);
for (const a of set) console.log(a); // 1, 2, 3

const map = new Map([
	["a", 1],
	["b", 2],
	["c", 3],
]);
for (const a of map) console.log(a); // ['a', 1], ['b', 2], ['c', 3]

console.log(arr[0]); // 1
console.log(set[0]); // undefined
console.log(map[0]); // undefined
```

Set과 Map의 for-of문은 어떻게 동작하고 있는 걸까요?

arr와 set, map의 내부를 모두 콘솔로 찍어보았습니다. arr는 key와 value로 이루어져있고, set과 map은 [[Entries]]라는 아이가 들어있습니다. [[Entries]]가 무엇인지, set과 map이 어떻게 for-of문을 돌고 있는 것인지 알려면 이터러블과 이터레이터에 대해 알아야 합니다(드디어 등장).
![functional-programming-2-1](/assets/images/functional-programming/functional-programming-2-1.png)

## 이터러블 (Iterable)

이터러블은 이터레이터를 리턴하며, [Symbol.iterator]()를 가진 값입니다.

읭?.. 싶긴 하지만 차근차근 읽다 보면 갑자기 이해되는 순간이 옵니다. 천천히 봐봅시다.

(참고로, Symbol은 문자열 대신 유니크한 key를 생성하기 위한 도구입니다. 개발자들끼리 약속한 키를 만들 때 유용합니다.)

Array는 이터러블입니다. Array는 이터레이터를 리턴하며, [Symbol.iterator]()를 가졌기 때문입니다. 확인해보고 싶다면 콘솔을 찍어보면 됩니다. arr[Symbol.iterator]()를 찍어보면 { [ Iterater ] }를 리턴합니다. (참고로, Set과 Map도 이터러블 입니다. 예시로 Array를 먼저 보겠습니다.)

```javascript
console.log(arr[Symbol.iterator]()); // {[Iterater]}
```

이번엔 arr[Symbol.iterator]에 null 값을 할당해봅시다. 그러면 for-of문은 돌지 못하고 arr가 iterable이 아니라는 에러가 나옵니다. 앗, 이터러블이 아니면 for-of문을 돌 수 없나봅니다.

```javascript
const arr = [1, 2, 3];
arr[Symbol.iterator] = null;
for (const a of arr) console.log(a); // arr is not iterable ..
```

그렇다면 arr를 es5의 리스트 순회 방식으로 돌리면 어떨까요? arr[Symbol.iterator]에 null을 할당해줘도 결괏값을 잘 리턴합니다. 이유는 위에서 언급했듯 Array는 key와 value의 쌍으로 이루어졌기 때문에 우리가 잘 알듯, 익숙하게 반복문을 돌릴 수 있습니다.

```javascript
const arr = [1, 2, 3];
arr[Symbol.iterator] = null;
// for (const a of arr) console.log(a);
for (var i = 0; i < arr.length; i++) {
	console.log(arr[i]); // 1, 2, 3
}
```

## 이터레이터 (Iterator)

이터레이터는 {value, done} 객체를 리턴하며, next()를 가진 값입니다.

역시나 읭, 스럽긴 한데 천천히 읽어봅시다.

이터러블은 arr[Symbol.iterator]를 리턴하며 이터레이터를 리턴한다고 했습니다. 그럼 리턴한 이터레이터를 iterator라는 변수에 담아봅시다. 그리고 interator.next()를 찍어보니 이터레이터의 정의대로 {value, done} 객체를 리턴합니다. 변수에 담은 iterator는 이터레이터가 맞나 봅니다.

```javascript
const arr = [1, 2, 3];
let iterator = arr[Symbol.iterator]();

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```

## 이터러블/이터레이터 프로토콜

이터러블/이터레이터 프로토콜은 이터러블을 for-of, 전개 연산자 등과 함께 동작하도록 한 규약을 말합니다.

이터ㄹㅂ..지옥에 빠진 것 같습니다. 그래도 포기하지 말고 읽어봅시다(스스로에게 하는 말,,).

이터러블인 Array와 Set, Map은 이터러블/이터레이터 프로토콜을 따릅니다. 즉, Array와 Set, Map는 이터러블/이터레이터 프로토콜에 따라 for-of으로 동작할 수 있다는 말이 되겠네요.

Set을 예제로 for-of문으로 어떻게 동작하는지 더 자세히 봐봅시다. 이터러블인 set이 리턴한 이터레이터를 iterator라는 변수에 담았습니다. 그리고 iterator.next()를 호출하면 value와 done이 담긴 객체가 리턴되는데 이때 value에 담긴 값(1, 2, 3)을 for-of문을 돌 때 하나씩 꺼내주다가 value가 undefined이며 done이 true가 되면 for-of문을 빠져나오게 됩니다. 이러한 과정으로 Set과 Map은 이터러블/이터레이터 프로토콜을 따르기 때문에 Array처럼 arr[0]으로 접근할 수 없더라도 for-of문으로 순회가 가능했던 것입니다.

```javascript
const set = new Set([1, 2, 3]);
let iterator = set[Symbol.iterator]();

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```

## Map의 keys(), values(), entries()

마지막으로, Map의 map.keys(), map.values(), map.entries()에 대해서 알아봅시다. 이 함수들은 모두 이터레이터를 리턴합니다. 이름처럼 keys()는 map의 key들을 반환하고, values()는 value들을, entries()는 전체를 반환합니다.

```javascript
const map = new Map([
	["a", 1],
	["b", 2],
	["c", 3],
]);
for (const a of map.keys()) console.log(a); // a, b, c
for (const a of map.values()) console.log(a); // 1, 2, 3
for (const a of map.entries()) console.log(a); // ["a", 1], ["b", 2], ["c", 3]
```

keys()를 예시로 map.keys()가 어떻게 key들을 반환하는지 확인해 봅시다. map.keys()를 담아 이터레이터인 변수 keysIterator를 next()로 찍어봅시다. 그럼 value에 map의 key값이 담겨 있습니다. 그러므로 map.keys()를 for-of문으로 돌리면 value에 담긴 값을 하나씩 꺼내 순회하기 때문에 key값을 얻을 수 있었습니다.
![functional-programming-2-2](../assets/images/functional-programming/functional-programming-2-2.png)

# 마무리

사실 연초에도 알고리즘을 공부하며 이터레이터에 대해서 공부했던 적이 있었는데, 이터ㄹㅂ..지옥에 빠졌었던 것 같습니다. 이번에는 함수형 프로그래밍을 배워보고싶다는 열망으로 계속 곱씹어 보았더니 이터러블, 이터레이터에 대해 조금은 감이 잡히는 것 같네요. 다음 글에서는 이터러블을 직접 만들어보면서 이터러블이 어떻게 생겼고, 어떻게 동작하는지 더 깊이 이해해봅시다. 😎

# 참고

- 유인동 님의 "함수형 프로그래밍과 Javascript ES6+" 강의를 바탕으로 작성했습니다.

- 강의를 들으며 저의 해석을 덧붙였습니다.
