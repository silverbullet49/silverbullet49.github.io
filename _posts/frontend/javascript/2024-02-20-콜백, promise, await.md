---
title: "콜백, promise, await"
categories:
  - study
tags:
  - html
  - css
  - javascript
  - typescript
---

# 콜백, promise, await

## 콜백 방식

```javascript
  const getMovies = (movieName, callback) => {
    fetch(`https://www.omdbapi.com/?apikey=7035c60c&s=${movieName}`)
    .then(res => res.json())
    .then(res => {
      console.log(res)
      callback()
    })
  }

  //  콜백 지옥..
  getMovies('fronzen', ()=>{
    console.log('겨울왕국')
    getMovies('avengers', ()=>{
      console.log('어벤저스')
      getMovies('avatar', ()=>{
        console.log('아바타')
      })
    })
  })
```

## promise
- 콜백 지옥 해결 시도

```javascript
  const getMovies = (movieName) => {
    return new Promise(resolve => {
      fetch(`https://www.omdbapi.com/?apikey=7035c60c&s=${movieName}`)
          .then(res => res.json())
          .then(res => {
            console.log(res)
            resolve()
          })
    })
  }

  getMovies('fronzen')
    .then(() => {
      console.log('겨울왕국')
      return getMovies('avengers')
    })
    .then(() => {
      console.log('어벤저스')
      return getMovies('avatar')
    })
    .then(() => {
        console.log('아바타')
    })
```

## await / async 방법

- promise를 키워드로 대체.
- await를 통해 함수의 종료를 기다림.
- await 함수는 promise를 리턴해야 함.
- await를 쓰려면 async 함수로 감싸야 함.

```javascript
  const getMovies = (movieName) => {
    return new Promise(resolve => {
      fetch(`https://www.omdbapi.com/?apikey=7035c60c&s=${movieName}`)
          .then(res => res.json())
          .then(res => {
            console.log(res)
            resolve()
          })
    })
  }


  const wrap = async () => {
    await getMovies('fronzen')
    console.log('겨울왕국')
    await getMovies('avengers')
    console.log('어벤저스')
    await getMovies('avatar')
    console.log('아바타')
  }
  wrap()

```

## 예외 처리

### promise

```javascript
const delayAdd = index => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (index > 5) {
        reject('reject!')
        return
      }
      resolve(index + 1);
    }, 1000);
  });
}

// then을 스킵 하고 catch가 실행.
// index > 5일 때는 resolve가 스킵되고 reject가 실행되도록 작성했기 때문
delayAdd(6)
  .then(res => console.log(res))
  .catch(err => console.log(err))

```

### async / await

```javascript
const delayAdd = index => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (index > 5) {
        reject('reject!')
        return
      }
      resolve(index + 1);
    }, 1000);
  });
}

// await의 리턴 값은 resolve에 들어간 파라미터.
// reject 호출시 익셉션 발생. catch를 통해 처리.
const wrap = async () => {
  try {
    const p = await delayAdd(6)
    console.log(p)
  } catch (err) {
    console.log(err)
  }
}
wrap()

```

### 실험

- promise 콜백을 잘못 작성하여 reject, resolve 둘 다 실행했을 때는 어떻게 될까?
  - resolve, reject 중 먼저 실행된 놈이 결과를 가져간다.

```javascript
//reject가 먼저 실행되는 케이스

const delayAdd = index => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (index > 5) {
        reject('reject!')
        //return
      }
      resolve(index + 1);
    }, 1000);
  });
}

const wrap = async () => {
  try {
    //delayAdd(6) 호출시 promise 콜백이 곧바로 수행됨.(await랑 상관 없이)
    //index > 5이므로 reject 호출 후 resolve도 호출됨.
    const p = await delayAdd(6)
    console.log(p)
  } catch (err) {
    console.log(err)
  }
}
wrap() //결과: reject!
```


```javascript
//resolve가 먼저 실행되는 케이스

const delayAdd = index => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(index + 1);
      if (index > 5) {
        reject('reject!')
        //return
      }
    }, 1000);
  });
}

const wrap = async () => {
  try {
    //delayAdd(6) 호출시 promise 콜백이 곧바로 수행됨.(await랑 상관 없이)
    //index > 5이므로 resolve 호출 후 reject도 호출됨.
    const p = await delayAdd(6)
    console.log(p)
  } catch (err) {
    console.log(err)
  }
}
wrap() //결과: 7
```