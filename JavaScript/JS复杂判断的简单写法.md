# JS复杂判断的简单写法

一般在做逻辑判断的时候，常使用if...else来进行基本判断，但是随着条件越来越负责，逻辑if...else越来越臃肿，且难以读懂。
如下案例：
```js
const getResult = (status) => {
  if (status === 1) {
    console.log('status 1')
  } else if (status === 2) {
    console.log('status 2')
  } else if (status === 3) {
    console.log('status 3')
  }
}
```
这时候有人会提到，那就用switch...case。
```js
const getResult = (status) => {
  switch (status) {
    case 1:
      console.log('status 1')
      break
    case 2:
      console.log('status 2')
      break
    case 3:
      console.log('status 3')
      break
  }
}
```
但是今天这里要介绍另一种更简便的方式。
```js
const getResult = (status) => {
  const statusList = {
    '1': () => console.log('status 1'),
    '2': () => console.log('status 2'),
    '3': () => console.log('status 3'),
    'default': () => console.log('status is not found')
  }
  const fn = statusList[status] || statusList['default']
  fn()
}
```
但是假如需要判断的条件越来越多，无论是使用if...else还是switch...case，代码都会越来越臃肿，以至于难以维护，例如以下这种状况。
```js
const getMultiCondResult = (sex, status) => {
  if (sex === 'male') {
    if (status === 1) {
      console.log('male status 1')
    } else if (status === 2) {
      console.log('male status 2')
    } else if (status === 3) {
      console.log('male status 3')
    }
  } else if (sex === 'female') {
    if (status === 1) {
      console.log('female status 1')
    } else if (status === 2) {
      console.log('female status 2')
    } else if (status === 3) {
      console.log('female status 3')
    }
  }
}
```
这时候就需要一种更加简便的方式来优化下当前的代码了。
下面介绍一种更加直观的方式。
```js
const getMultiCondResult = (sex, status) => {
  const condList = {
    'male_1': () => console.log('male status 1'),
    'male_2': () => console.log('male status 1'),
    'male_3': () => console.log('male status 1'),
    'female_1': () => console.log('female status 1'),
    'female_2': () => console.log('female status 2'),
    'female_3': () => console.log('female status 3'),
    'default': () => console.log('cond is not found')
  }

  const key = `${sex}_${status}`
  const fn = condList[key] || condList['default']
  fn()
}
```
同时我们也可以使用es6的Map进行改写。
```js
const getMultiCondResult = (sex, status) => {
  const condMap = new Map([
    ['male_1', () => console.log('male status 1')],
    ['male_2', () => console.log('male status 2')],
    ['male_3', () => console.log('male status 3')],
    ['female_1', () => console.log('female status 1')],
    ['female_2', () => console.log('female status 2')],
    ['female_3', () => console.log('female status 3')],
    ['default', () => console.log('cond is not found')],
  ])

  const fn = condMap.get(`${sex}_${status}`) || condMap.get('default')
  fn()
}
```
当条件中有规律时，我们可以使用更加正则表达式，缩短代码量，也可以称之为进阶版。
如下，我们的代码越来越简洁，同时也可以很直观的看出条件的差别，减低了代码维护的难度。
```js
const getMultiCondResult = (...args) => {
  const arr = args.slice(0, 2)
  const innerArgs = args.slice(2)
  const condArr = [
    [/^(male|female)_[1-3]{1}$/, (a) => console.log(`${arr[0]} status ${arr[1]}`)]
  ]

  const type = arr.join('_')
  const action = [...condArr].filter(([key, value]) => key.test(type))
  action.forEach(([key, value]) => value.apply(this, innerArgs))
}
```
