# Action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

例子：

	const store = new Vuex.Store({
		state: {
			count: 0
		},
		mutations: {
			increment: function(state, playload) {
				state.count += playload.amount;
			}
		},
		actions: {
			increment(context, playload) {
				setTimeout(function () {
					context.commit({
						type: 'increment',
						amount: playload.amount
					});
				}, 1000)
			}
		}
	})

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象和一个载荷（playload）对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。

## 分发 Action

Action 通过 store.dispatch 方法触发：

	store.dispatch('increment')

Actions 支持和mutation同样的载荷方式和对象方式进行分发：

	// 以载荷形式分发
	store.dispatch('increment', {
		amount: 10
	})

	// 以对象形式分发
	store.dispatch({
		type: 'increment',
		amount: 10
	})

## 在组件中分发 Action

在组件中使用 this.$store.dispatch('xxx') 分发 action，或者使用 mapActions 辅助函数将组件的 methods 映射为 store.dispatch 调用（需要先在根节点注入 store）

	import { mapActions } from 'vuex'

	export default {
		// ...
		methods: {
			...mapActions([
				'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

				// `mapActions` 也支持载荷：
				'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
			]),
			...mapActions({
				add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
			})
		}
	}

## 组合 Action

### 使用promise 组合 Action

store.dispatch 可以处理被触发的 action 处理函数返回的 Promise，并且 store.dispatch 仍旧返回 Promise，因此可以组合多个 action，以处理复杂的异步流程。

	actions: {
		actionA ({ commit }) {
			return new Promise((resolve, reject) => {
				setTimeout(() => {
					commit('someMutation')
					resolve()
				}, 1000)
			})
		}
	}

然后可以在分发action时使用`.then()`执行后续操作：

	store.dispatch('actionA').then(() => {
		// ...
	})

在另外一个 action 中也可以：

	actions: {
		// ...
		actionB ({ dispatch, commit }) {
			return dispatch('actionA').then(() => {
				commit('someOtherMutation')
			})
		}
	}

### 利用 async / await 组合 Action

	// 假设 getData() 和 getOtherData() 返回的是 Promise

	actions: {
		async actionA ({ commit }) {
			commit('gotData', await getData())
		},
		async actionB ({ dispatch, commit }) {
			await dispatch('actionA') // 等待 actionA 完成
			commit('gotOtherData', await getOtherData())
		}
	}

一个 store.dispatch 在不同模块中可以触发多个 action 函数。在这种情况下，只有当所有触发函数完成后，返回的 Promise 才会执行。???


