# Async/await

편리한 방식으로 프라미스(promise)를 다루는 특별한 구문을 "async/await"이라고 합니다. 이것은 놀라울 정도로 사용하기 쉽고 이해하기도 쉽습니다.

## Async functions

`async` 키워드부터 시작하겠습니다. 다음과 같이 함수 앞에 놓을 수 있습니다.

```js
async function f() {
  return 1;
}
```

함수 앞에 "async"라는 단어는 '함수는 항상 프라미스를 반환한다는 것'을 의미합니다. 다른 값들은 자동적으로 resolve된 프라미스에 래핑(wrapping)됩니다.

예를 들어, 이 함수는 1의 결과로 resolve된 프라미스를 반환합니다. 테스트 해보겠습니다.

```js run
async function f() {
  return 1;
}

f().then(alert); // 1
```

...우리는 명시적으로 프라미스를 반환할 수 있었습니다. 아래 함수는 위 함수와 동일합니다.

```js run
async function f() {
  return Promise.resolve(1);
}

f().then(alert); // 1
```

따라서 `async`는 프라미스를 반환하는 것을 보장하며 non-promise(프라미스가 아닌 객체)를 래핑합니다. 꽤 간단하지 않나요? 그러나 그것뿐만이 아닙니다. `async`함수 내에서만 작동하는 `await`이라는 키워드도 있습니다. 아주 멋진 키워드입니다.

## Await

문법:

```js
// async 함수에서만 작동합니다.
let value = await promise;
```

`await`키워드는 프라미스가 처리되고 결과를 반환할 때까지 자바스크립트를 대기시킵니다.

여기 1초 안에 resolve되는 프라미스 예시가 있습니다:
```js run
async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

*!*
  let result = await promise; // 프라미스가 resolve될 때까지 기다리세요 (*)
*/!*

  alert(result); // "done!"
}

f();
```

함수를 실행하면 `(*)` 표시가 되어있는 행에서 "일시정지"되고 프라미스가 처리되면 `result`에 할당됩니다. 그래서 위의 코드는 1초 안에 "done!"을 보여줍니다.

중요: `await` 은 문자 그대로 프라미스가 처리될 때까지 자바스크립트를 기다리게 만든 후 다음 결과를 진행합니다. 이것은 엔진이 다른 작업을 수행할 수 있기 때문에 CPU 리소스 비용이 들지 않습니다: 다른 스크립트 실행, 이벤트 처리 등.

이것은 프라미스의 결과를 얻을 수 있는 `promise.then`보다 우아한 구문이며, 읽고 쓰기 쉽습니다. 

````warn header="보편적인 정규 함수식에서는 `await`을 쓸 수 없습니다"
만약 non-async(비동기) 함수에서 `await`을 쓴다면,구문 오류가 발생합니다:

```js run
function f() {
  let promise = Promise.resolve(1);
*!*
  let result = await promise; // 구문 오류
*/!*
}
```

함수 앞에 `async` 를 쓰지 않으면 오류가 발생합니다. 말했듯이, `await` 은 `async function` 내에서만 작동합니다.
````

 <info:promise-chaining>에서 `showAvatar()`예제를 가져와 `async/await`을 사용하여 다시 작성해보겠습니다:

1. `.then`을 `await`으로 바꿔야 합니다.
2. 또한 함수가 작동하도록 `async`로 만들어야 합니다.

```js run
async function showAvatar() {

  // read our JSON
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();

  // read github user
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // show the avatar
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // wait 3 seconds
  await new Promise((resolve, reject) => setTimeout(resolve, 3000));

  img.remove();

  return githubUser;
}

showAvatar();
```

꽤 깔끔하고 읽기 쉽습니다. 그렇지 않나요? 전보다 훨씬 낫습니다.

````smart header="`await` 은 최상위 코드에서 작동하지 않습니다."
`await`을 처음 써본 사람들은 최상위 코드(top-level code)에서 `await`을 사용할 수 없다는 걸 까먹는 경향이 있습니다. 예를 들어, 다음과 같이 작동하지 않습니다.

```js run
// 최상위 코드에서의 구문 오류
let response = await fetch('/article/promise-chaining/user.json');
let user = await response.json();
```

우리는 다음과 같이 익명의 비동기 함수(async function))로 래핑할 수 있습니다:

```js run
(async () => {
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();
  ...
})();
```
// 여기까지. 그냥 여 줄만 지우고 시작. 밑에 것 어려움. 

````
````smart header="`await` accepts \"thenables\""
`promise.then`과 같이 `await` 은 thenable 객체(`then` 메서드를 호출할 수 있는 객체)를 쓸 수 있도록 해줍니다. 이 의미는 서드 파티 객체가 프라미스가 아니라도 프라미스와 호환될 수 있다는 것입니다: 만약 그 객체가 `.then`을 지원한다면, `await` 을 쓰기에 충분합니다.

여기 `Thenable` 클래스 데모가 있습니다. `await` 아래에 인스턴스를 허락한다:

```js run
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve);
    // resolve with this.num*2 after 1000ms
    setTimeout(() => resolve(this.num * 2), 1000); // (*)
  }
};

async function f() {
  // waits for 1 second, then result becomes 2
  let result = await new Thenable(1);
  alert(result);
}

f();
```

If `await` gets a non-promise object with `.then`, it calls that method providing native functions `resolve`, `reject` as arguments. Then `await` waits until one of them is called (in the example above it happens in the line `(*)`) and then proceeds with the result.
````

````smart header="Async class methods"
To declare an async class method, just prepend it with `async`:

```js run
class Waiter {
*!*
  async wait() {
*/!*
    return await Promise.resolve(1);
  }
}

new Waiter()
  .wait()
  .then(alert); // 1
```
The meaning is the same: it ensures that the returned value is a promise and enables `await`.

````
## Error handling

If a promise resolves normally, then `await promise` returns the result. But in case of a rejection, it throws the error, just as if there were a `throw` statement at that line.

This code:

```js
async function f() {
*!*
  await Promise.reject(new Error("Whoops!"));
*/!*
}
```

...Is the same as this:

```js
async function f() {
*!*
  throw new Error("Whoops!");
*/!*
}
```

In real situations, the promise may take some time before it rejects. In that case there will be a delay before `await` throws an error.

We can catch that error using `try..catch`, the same way as a regular `throw`:

```js run
async function f() {

  try {
    let response = await fetch('http://no-such-url');
  } catch(err) {
*!*
    alert(err); // TypeError: failed to fetch
*/!*
  }
}

f();
```

In case of an error, the control jumps to the `catch` block. We can also wrap multiple lines:

```js run
async function f() {

  try {
    let response = await fetch('/no-user-here');
    let user = await response.json();
  } catch(err) {
    // catches errors both in fetch and response.json
    alert(err);
  }
}

f();
```

If we don't have `try..catch`, then the promise generated by the call of the async function `f()` becomes rejected. We can append `.catch` to handle it:

```js run
async function f() {
  let response = await fetch('http://no-such-url');
}

// f() becomes a rejected promise
*!*
f().catch(alert); // TypeError: failed to fetch // (*)
*/!*
```

If we forget to add `.catch` there, then we get an unhandled promise error (viewable in the console). We can catch such errors using a global event handler as described in the chapter <info:promise-error-handling>.


```smart header="`async/await` and `promise.then/catch`"
When we use `async/await`, we rarely need `.then`, because `await` handles the waiting for us. And we can use a regular `try..catch` instead of `.catch`. That's usually (not always) more convenient.

But at the top level of the code, when we're outside of any `async` function, we're syntactically unable to use `await`, so it's a normal practice to add `.then/catch` to handle the final result or falling-through errors.

Like in the line `(*)` of the example above.
```

````smart header="`async/await` works well with `Promise.all`"
When we need to wait for multiple promises, we can wrap them in `Promise.all` and then `await`:

```js
// wait for the array of results
let results = await Promise.all([
  fetch(url1),
  fetch(url2),
  ...
]);
```

In case of an error, it propagates as usual: from the failed promise to `Promise.all`, and then becomes an exception that we can catch using `try..catch` around the call.

````

## Summary

The `async` keyword before a function has two effects:

1. Makes it always return a promise.
2. Allows to use `await` in it.

The `await` keyword before a promise makes JavaScript wait until that promise settles, and then:

1. If it's an error, the exception is generated, same as if `throw error` were called at that very place.
2. Otherwise, it returns the result.

Together they provide a great framework to write asynchronous code that is easy both to read and write.

With `async/await` we rarely need to write `promise.then/catch`, but we still shouldn't forget that they are based on promises, because sometimes (e.g. in the outermost scope) we have to use these methods. Also `Promise.all` is a nice thing to wait for many tasks simultaneously.
