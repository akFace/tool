<template>

	<div>

		<div class="head">

			<input ref="input" class="input" placeholder="请输入用户MID / 账号进行搜索" return-key-type="search" />

			<icon class="icon" content="tb-search" />

		</div>

		<div class="main" @touchend="end" @touchmove="move" @touchstart="start">

			<text class="item" v-for="item in array" :key="item.t" :ref="item.t" :style="{ transform: `translate(${item.x},${item.y})` }">{{ item.t }}</text>

		</div>

	</div>

</template>

<style>

	.head {
		width: 750;
		height: 100;
		align-items: center;
		flex-direction: row;
		background-color: #191919;
	}

	.icon {
		width: 100;
		height: 100;
	}

	.input {
		color: #EEE;
		width: 630;
		height: 60;
		padding: 0 20;
		font-size: 25;
		margin-left: 20;
		border-radius: 10;
		background-color: #222;
	}

	.main {
		flex: 1;
		width: 750;
	}

	.item {
		top: 0;
		left: 0;
		width: 162.5;
		font-size: 25;
		line-height: 60;
		border-radius: 10;
		color: #FFF;
		position: absolute;
		text-align: center;
		background-color: #04CC66;
	}

</style>

<script>

	// 不支持动态绑定css transition-duration属性 所以依赖animation模块
	const { transition } = app.requireModule('animation')

	export default {

		data: () => ({
			index: null,	// 当前标签索引
			array: [		// 标签数组 x x轴坐标 y y轴坐标 因为weex宽为750 所以大家可以根据自己的标签的宽高和间距来自己定义
				{ t: 'c', x: 20, y: 20 },
				{ t: 'c#', x: 202.5, y: 20 },
				{ t: 'c++', x: 385, y: 20 },
				{ t: 'go', x: 567.5, y: 20 },
				{ t: 'java', x: 20, y: 100 },
				{ t: 'nodejs', x: 202.5, y: 100 },
				{ t: 'python', x: 385, y: 100 },
				{ t: 'php', x: 567.5, y: 100 },
				{ t: 'javascript', x: 20, y: 180 },
				{ t: 'typescript', x: 202.5, y: 180 },
				{ t: 'html', x: 385, y: 180 },
				{ t: 'css', x: 567.5, y: 180 }
			]
		}),

		methods: {
			// 设置当前标签
			set () {
				// 根据当前标签索引判断xy坐标
				let x = this.index % 4 * 182.5 + 20
				let y = Math.floor(this.index / 4) * 80 + 20
				let [el] = this.$refs[this.array[this.index].t]
				// 进行动画
				this.transition(x, y, el, () => {
					// 因为异步 防止多手指触摸 第一根手指松开后 其余手指再松开 此时index=null 从而this.array[null].x 不是一个对象而报错
					if (this.index == null) return
					// 等待动画完毕后 更新xy坐标
					this.array[this.index].x = x
					this.array[this.index].y = y
					// 清除当前标签索引
					this.index = null
				})
			},

			end (e) {
				// 如果一开始你啥都没摸到 直接return
				if (this.index == null) return
				// 声明边界变量
				let t = 0
				let l = 0
				let r = 750
				let b = 80 * Math.floor(this.array.length / 4)
				let [{pageX, pageY}] = e.changedTouches
				// 边界判断
				if (pageY > t && pageY < b && pageX > l && pageX < r) {
					l = 162.5 / 2 + 20
					let x = 0
					// 判断你要挪动到第几行
					let y = Math.floor(pageY / 80)
					// 两个标签之间的宽 因为四个标签有三个间隙 所以两个标签之间的宽为 半个标签宽 * 2 + 20
					let w = (750 - l * 2) / (4 - 1)

					if (pageX < l) {
						// 当触摸坐标小于 半个标签宽 + 边距时，说明你要挪动到最左边
						x = 0
					} else if (pageX > 750 - l) {
						// 当触摸坐标大于 屏幕宽 - (半个标签宽 + 边距)时，说明你要挪动到最右边
						x = 4
					} else {
						// 我们根据(触摸坐标 - (半个标签宽 + 边距)) / 两个标签之间的宽 来判断你要挪到第几列
						x = Math.ceil((pageX - l) / w)
					}
					// 缓存最后更换的标签变量
					let end = null
					// 根据上面我们求出的行列算出被替换标签索引
					let index = x + y * 4
					// 这说明我们往后挪动了
					if (index > this.index) {
						// 因为往后挪动 所以我们的index必须-- 如果--后大于了当前标签索引 那就说明挪动了
						if (--index > this.index) {
							// 缓存一下当前标签 因为obj引用类型 大伙懂的都得
							end = this.array[this.index]
							// 循环从当前标签到被替换标签之间的所有标签 他们要依次往前走
							for (let i = this.index; i < index; i++) {
								// 更新索引
								this.array[i] = this.array[i + 1]
								// 计算xy坐标
								let x = i % 4 * 182.5 + 20
								let y = Math.floor(i / 4) * 80 + 20
								let [el] = this.$refs[this.array[i].t]
								// 开始动画
								this.transition(x, y, el, () => {
									this.array[i].x = x
									this.array[i].y = y
								})
							}
							// 循环完成后 我们把替换的标签索引 换成我们当前的缓存的 因为循环过程中this.index指向的数据已被更新
							this.array[index] = end
							// 并且更新当前标签索引
							this.index = index
						}
						// 这说明我们往前挪动了
					} else if (index < this.index) {
						// 跟上面一样 我懒得封装了。。。 只不过这次是往前移动 所以不用--了
						end = this.array[this.index]
						for (let i = this.index; i > index; i--) {
							this.array[i] = this.array[i - 1]
							let x = i % 4 * 182.5 + 20
							let y = Math.floor(i / 4) * 80 + 20
							let [el] = this.$refs[this.array[i].t]
							this.transition(x, y, el, () => {
								this.array[i].x = x
								this.array[i].y = y
							})
						}
						this.array[index] = end
						this.index = index
					}
				}
				// 更新当前标签xy坐标
				this.set()
			},

			// 正在移动
			move (e) {
				// 如果一开始你啥都没摸到 直接return
				if (this.index == null) return
				// 获取触摸坐标并更新当前标签的xy坐标
				let [{pageX, pageY}] = e.changedTouches
				this.array[this.index].x = pageX - 162.5 / 2
				this.array[this.index].y = pageY - 60 / 2
			},

			// 开始移动
			start (e) {
				if (this.index) return
				// 获取拖动的坐标
				let [{pageX, pageY}] = e.changedTouches
				// 如果为margin区域 啥都不做 因为我margin是20 所以写死了
				if (pageX % 182.5 < 20 || pageY % 80 < 20) return
				// 根据触摸坐标来判断行列并算出触摸标签索引
				let index = Math.floor(pageX / 182.5) + Math.floor(pageY / 80) * 4
				// 如果 this.array[index] == undefined 说明触摸行列没用标签
				if (this.array[index] == undefined) return
				// 如果存在 更新当前标签索引
				this.index = index
				this.array[this.index].x = pageX - 162.5 / 2
				this.array[this.index].y = pageY - 60 / 2
			},

			// 动画函数
			transition (x, y, el, fn) {
				transition(el, {
					styles: { transform: `translate(${x},${y})` },
					duration: 100
				}, () => { if (fn) fn() })
			}
		}

	}

</script>
