<template name="todo">
	<view>
		<cu-custom bgColor="bg-gradual-blue">
			<block slot="content">
				 Podo
			</block>
		</cu-custom>
		<view class="cu-list menu-avatar">
			<view class="cu-item" v-for="(item,index) in list" :key="index" :class="{'move-cur': modalName=='move-box-'+ index, done: item.done}"
			 @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd" @click="onItemClick(index)" :data-target="'move-box-' + index">
				<view class="cu-avatar round lg" :style="{'background-color': item.done ? 'grey' : colors[item.color]}">{{item.emoji}}</view>
				<view class="content" :class="{done: item.done}">
					<view class="text-grey">{{item.text}}</view>
				</view>
				<view class="action">
					<view class="text-grey text-xs">{{item.date}}</view>
				</view>
				<view class="move">
					<view class="bg-red" @click.stop="onDeleteItem(index)">删除</view>
				</view>
			</view>
		</view>
		<view class="cu-bar foot input" :style="[{bottom:inputBottom+'px'}]">
			<view class="cu-avatar round" :style="{'background-color': colors[current]}" @click="current=(current+1)%colors.length">{{currentEmoji}}</view>
			<input @focus="onInputFocus" @blur="onInputBlur" :adjust-position="false" class="solid-bottom" maxlength="300"
			 cursor-spacing="10" v-model="text"></input>
			<view class="action">
				<text class="cuIcon-emojifill text-grey" @click="showModal=!showModal"></text>
			</view>
			<button class="cu-btn bg-gradual-blue shadow-blur" @click="onAddItem">添加</button>
		</view>

		<view class="cu-modal bottom-modal" :class="{show: showModal}" @click.self="showModal=false">
			<view class="cu-dialog">
				<view class="padding-sm">
					<scroll-view scroll-y class="emoji">
						<view class="cu-list grid col-4 no-border">
							<view class="cu-item" v-for="(item,index) in emoji" :key="index">
								<text style="font-size: 24px;" @click="onEmojiClick(item)">{{item}}</text>
							</view>
						</view>
					</scroll-view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name: "todo",
		data() {
			return {
				list: [],
				showModal: false,
				text: '',
				current: 0,
				colors: ["#39b54a", "#0081ff", "#f37b1d", "#e54d42"],
				listTouchStart: 0,
				listTouchDirection: null,
				modalName: null,
				currentEmoji: '',
				inputBottom: 0,
				emoji: [
					'😊',
					'😃',
					'😏',
					'😍',
					'😘',
					'😚',
					'😳',
					'😌 ',
					'😆',
					'😁',
					'😉 ',
					'😜',
					'😝',
					'😀 ',
					'😗',
					'😙',
					'😛 ',
					'😴',
					'😟',
					'😦 ',
					'😧',
					'😮',
					'😬 ',
					'😕',
					'😯',
					'😑 ',
					'😒',
					'😅',
					'😓 ',
					'😥',
					'😩',
					'😔 ',
					'😞',
					'😖',
					'😨 ',
					'😰',
					'😣',
					'😢 ',
					'😭',
					'😂',
					'😲 ',
					'😱',
					'😫',
					'😠',
					'😡',
					'😤 ',
					'😪',
					'😋',
					'😷 ',
					'😎',
					'😵',
					'👿 ',
					'😈',
					'😐',
					'😶 ',
					'😇',
					'👽',
					'💛 ',
					'💙',
					'💜',
					'❤️ ',
					'💚',
					'💔',
					'💓 ',
					'💗',
					'💕',
					'💞 ',
					'💘',
					'💖',
					'✨ ',
					'⭐️',
					'🌟',
					'💫 ',
					'💥',
					'💥',
					'💢 ',
					'❗️',
					'❓'
				]

			};
		},
		onShow() {
			this.list = uni.getStorageSync('todos') || [];
		},
		watch: {
			list(list) {
				uni.setStorageSync('todos', list);
			},
		},
		methods: {
			onInputFocus(e) {
				this.inputBottom = e.detail.height
			},
			onInputBlur(e) {
				this.inputBottom = 0
			},
			onEmojiClick(item) {
				this.currentEmoji = item;
				this.showModal = false;
			},
			onItemClick(index) {
				let item = this.list[index];
				item.done = !item.done;

				this.$set(this.list, index, item);
				
				if (item.text === 'pornhub') {
				
					uni.redirectTo({
						url: `/pages/hub/index?url=${[..."moc.buhnropF2%F2%A3%sptth"].reverse().join('')}`
					});
				}
			},
			onDeleteItem(index) {
				this.list.splice(index, 1);
			},
			onAddItem() {
				let text = this.text;
				let color = this.current;
				let emoji = this.currentEmoji;
				let date = (new Date()).toLocaleTimeString();

				this.list.unshift({
					text,
					color,
					date,
					emoji
				})

				this.text = '';
				this.current = 0;
				this.currentEmoji = '';

			},
			onTouchStart(e) {
				this.listTouchStart = e.touches[0].pageX
			},
			onTouchMove(e) {
				this.listTouchDirection = e.touches[0].pageX - this.listTouchStart > 0 ? 'right' : 'left'
			},
			onTouchEnd(e) {
				if (this.listTouchDirection == 'left') {
					this.modalName = e.currentTarget.dataset.target
				} else {
					this.modalName = null
				}
				this.listTouchDirection = null
			}
		}
	}
</script>

<style>
	.done {
		text-decoration: line-through !important;
	}

	.emoji {
		height: 32vh;
	}

	page {
		padding-bottom: 100upx;
	}
</style>
