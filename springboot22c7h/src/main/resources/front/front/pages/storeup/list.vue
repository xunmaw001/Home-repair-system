<template>
<mescroll-uni @init="mescrollInit" :up="upOption" :down="downOption" @down="downCallback" @up="upCallback">
	<view class="content">
		<view :style='{"padding":"40rpx 0","borderColor":"#21d5ae","alignItems":"flex-start","display":"block","justifyContent":"space-between","minHeight":"100vh","flexWrap":"wrap","background":"url() fixed,#fff","borderWidth":"0px 0","width":"100%","position":"relative","borderStyle":"dashed","height":"auto"}'>
			<view class="cu-bar bg-white search" style="width:100%" :style="[{top:CustomBar + 'px'}]">
				<picker v-if="queryList.length>1" mode="selector" :range="queryList" range-key="queryName" :value="queryIndex" @change="queryChange" style="padding-left: 20upx;">
					<view><image style="width: 20upx;height: 33upx;" src="../../static/center/to.png"></image></view>
				</picker>
					<view v-if="queryIndex==0" class="search-form round">
						<text class="cuIcon-search"></text>
						<input v-model="searchForm.name" type="text" placeholder="名称" ></input>
					</view>
				<view class="action">
					<button @tap="search" class="cu-btn shadow-blur round">搜索</button>
				</view>
			</view>
			<view :style='{"width":"100%","alignItems":"flex-start","flexWrap":"wrap","background":"none","display":"flex","height":"auto"}'>

				<view class="uni-product-list" :style='{"padding":"0px 24rpx 24rpx","margin":"24rpx 0 0","flexWrap":"wrap","background":"none","flex":"1","display":"flex","width":"calc(100% - 220rpx)","justifyContent":"space-between","height":"auto"}'>
					<view @tap="onDetailTap(product)" class="uni-product" :style='{"padding":"20rpx","boxShadow":"inset 0px 0px 112rpx 0px #ecf2f9,0px 12rpx 18rpx -12rpx #ccc","margin":"0 0 32rpx","borderColor":"#f0d9da #c8d9eb #f0d9da","display":"flex","borderRadius":"40rpx","flexWrap":"wrap","borderWidth":"2rpx","background":"#fff","width":"100%","position":"relative","borderStyle":"solid","height":"auto"}' v-for="(product,index) in list" :key="index">
						<view class="uni-product-title " :style='{"padding":"4rpx 20rpx","margin":"0 0 8rpx","whiteSpace":"nowrap","color":"#333","textAlign":"center","overflow":"hidden","borderRadius":"20%","background":"none","width":"96%","lineHeight":"48rpx","fontSize":"28rpx","textOverflow":"ellipsis","order":"2"}'>{{product.name}}</view>
						<image :style='{"padding":"0","margin":"8rpx auto 8rpx","objectFit":"cover","borderRadius":"20rpx","display":"block","width":"100%","height":"240rpx","order":"1"}' mode="aspectFill" class="uni-product-image" :src="baseUrl+product.picture"></image>
					</view>
				</view>
			

			</view>
			
			
			
		</view>

		<button :style='{"border":"2rpx solid #fff","boxShadow":"0 4rpx 8rpx rgba(0,0,0,.1)","color":"#333","bottom":"120rpx","right":"128rpx","outline":"none","borderRadius":"100%","background":"#c8d9eb","width":"88rpx","lineHeight":"88rpx","fontSize":"28rpx","position":"fixed","height":"88rpx","zIndex":"9999"}' v-if="userid && isAuth('storeup','新增')" class="add-btn" @click="onAddTap()">新增</button>
		<button :style='{"border":"2rpx solid #fff","boxShadow":"0 4rpx 8rpx rgba(0,0,0,.1)","color":"#333","bottom":"120rpx","right":"128rpx","outline":"none","borderRadius":"100%","background":"#c8d9eb","width":"88rpx","lineHeight":"88rpx","fontSize":"28rpx","position":"fixed","height":"88rpx","zIndex":"9999"}' v-if="!userid && isAuthFront('storeup','新增')" class="add-btn" @click="onAddTap()">新增</button>
	</view>
</mescroll-uni>
</template>

<script>
	export default {
		data() {
			return {
				btnColor: ['#409eff','#67c23a','#909399','#e6a23c','#f56c6c','#356c6c','#351c6c','#f093a9','#a7c23a','#104eff','#10441f','#a21233','#503319'],
				queryList:[
					{
						queryName:"名称",
					},
				],
				queryIndex: 0,
				list: [],
				lists: [],
                userid: '',
                type: 1,
				mescroll: null, //mescroll实例对象
				downOption: {
					auto: false //是否在初始化后,自动执行下拉回调callback; 默认true
				},
				upOption: {
					noMoreSize: 5, //如果列表已无数据,可设置列表的总数量要大于半页才显示无更多数据;避免列表数据过少(比如只有一条数据),显示无更多数据会不好看; 默认5
					textNoMore: '~ 没有更多了 ~',
				},
				hasNext: true,
				searchForm:{},
				CustomBar: '0'
			};
		},
		computed: {
			baseUrl() {
				return this.$base.url;
			}
		},
		async onShow() {
			this.btnColor = this.btnColor.sort(()=> {
                                return (0.5-Math.random());
                        });
			this.hasNext = true
			// 重新加载数据
			if (this.mescroll) this.mescroll.resetUpScroll()
		},
		onLoad(options) {
            if(options.userid) {
                this.userid=options.userid;
            } else {
                this.userid = "";
            }
            if(options.menuJump) {
                this.type=options.menuJump;
            } else {
                this.type = 1;
            }
			this.hasNext = true
			// 重新加载数据
			if (this.mescroll) this.mescroll.resetUpScroll()
		},
		methods: {
            priceChange(price) {
                return Number(price).toFixed(2);
            },
            preHttp(str) {
                return str && str.substr(0,4)=='http';
            },
			//查询条件切换
			queryChange(e) {
				this.queryIndex=e.detail.value;
				this.searchForm.name="";
			},
			//类别搜索
			// mescroll组件初始化的回调,可获取到mescroll对象
			mescrollInit(mescroll) {
				this.mescroll = mescroll;
			},
			/*下拉刷新的回调 */
			downCallback(mescroll) {
				this.hasNext = true
				// 重置分页参数页数为1
				mescroll.resetUpScroll()
			},
			/*上拉加载的回调: mescroll携带page的参数, 其中num:当前页 从1开始, size:每页数据条数,默认10 */
			async upCallback(mescroll) {
				let params = {
					page: mescroll.num,
					limit: mescroll.size
				}


				if(this.searchForm.name){
					params['name'] = '%' + this.searchForm.name + '%'
				}


				params['type']=this.type;

                let res = {}
                if(this.userid) {
                    res = await this.$api.page(`storeup`, params);
                } else {
                    res = await this.$api.list(`storeup`, params);
                }
				// 如果是第一页数据置空
				if (mescroll.num == 1) this.list = [];
				this.list = this.list.concat(res.data.list);
				let length = Math.ceil(this.list.length/6)
				let arr = [];
				for (let i = 0; i<length; i++){
					arr[i] = this.list.slice(i*6, (i+1)*6)
				}
				this.lists = arr
				if (res.data.list.length == 0) this.hasNext = false;
				mescroll.endSuccess(mescroll.size, this.hasNext);
			},
			// 详情
			onDetailTap(item) {
				this.$utils.jump(`../${item.tablename}/detail?id=${item.refid}`)
			},
			// 修改
			onUpdateTap(id){
                uni.setStorageSync("useridTag",this.userid);
				this.$utils.jump(`./add-or-update?id=${id}`)
			},
			// 添加
			onAddTap(){
                uni.setStorageSync("useridTag",this.userid);
				this.$utils.jump(`./add-or-update`)
			},
			onDeleteTap(id){
				var _this = this;
				uni.showModal({
					title: '提示',
					content: '是否确认删除',
					success: async function(res) {
						if (res.confirm) {
							await _this.$api.del('storeup', JSON.stringify([id]));
							_this.hasNext = true
							// 重置分页参数页数为1
							_this.mescroll.resetUpScroll()
						}
					}
				});
			},
			// 搜索
			async search(){
				this.mescroll.num = 1
				let searchForm = {
					page: this.mescroll.num,
					limit: this.mescroll.size
				}
				if(this.searchForm.name){
					searchForm['name'] = '%' + this.searchForm.name + '%'
				}
				searchForm['type']=this.type;
                let res = {};
                if(this.userid) {
                    res = await this.$api.page(`storeup`, searchForm);
                } else {
                    res = await this.$api.list(`storeup`, searchForm);
                }
				// 如果是第一页数据置空
				if (this.mescroll.num == 1) this.list = [];
				this.list = this.list.concat(res.data.list);
				let length = Math.ceil(this.list.length/6)
				let arr = [];
				for (let i = 0; i<length; i++){
					arr[i] = this.list.slice(i*6, (i+1)*6)
				}
				this.lists = arr
				if (res.data.list.length == 0) this.hasNext = false;
				this.mescroll.endSuccess(this.mescroll.size, this.hasNext);
			}
		}
	};
</script>

<style lang="scss" scoped>
	.content {
		min-height: calc(100vh - 44px);
		box-sizing: border-box;
	}
	
	.category-one .tab {
		cursor: pointer;
		padding: 0 20rpx 24rpx;
		margin: 0 8rpx;
		color: #333;
		display: inline-block;
		font-size: 28rpx;
		border-color: #069ffa;
		line-height: 160rpx;
		border-radius: 0;
		background: url(http://codegen.caihongy.cn/20221220/f084fc07fcb643b8b92acdba4752de67.png) no-repeat left top,url(http://codegen.caihongy.cn/20221220/1e7abd511d1147b8afeb6858d54a7aef.png) no-repeat right top,url(http://codegen.caihongy.cn/20221220/d9457110e9a34ee49c5994b567d884a9.png) repeat-x;
		width: auto;
		border-width: 0;
		border-style: solid;
		text-align: center;
		min-width: 126rpx;
		height: 122rpx;
	}
	
	.category-one .tab.active {
		cursor: pointer;
		padding: 0 20rpx;
		margin: 0 8rpx;
		color: #333;
		font-weight: 600;
		display: inline-block;
		font-size: 28rpx;
		border-color: #069ffa;
		line-height: 160rpx;
		border-radius: 0;
		background: url(http://codegen.caihongy.cn/20221220/f084fc07fcb643b8b92acdba4752de67.png) no-repeat left top,url(http://codegen.caihongy.cn/20221220/1e7abd511d1147b8afeb6858d54a7aef.png) no-repeat right top,url(http://codegen.caihongy.cn/20221220/d9457110e9a34ee49c5994b567d884a9.png) repeat-x;
		width: auto;
		border-width: 0 0 0px;
		border-style: solid;
		text-align: center;
		min-width: 126rpx;
		height: 122rpx;
	}
	
	.category-two .tab {
		cursor: pointer;
		padding: 0;
		margin: 0 0 20rpx;
		color: #555;
		display: inline-block;
		font-size: 28rpx;
		border-color: #eee #ddd #ccc;
		line-height: 80rpx;
		border-radius: 20rpx;
		box-shadow: 0px 12rpx 18rpx -12rpx #999;
		background: linear-gradient(180deg, rgba(255,255,255,1) 0%, rgba(249,236,236,1) 100%);
		width: 100%;
		border-width: 2rpx;
		border-style: solid;
		text-align: center;
		height: 80rpx;
	}
	
	.category-two .tab.active {
		cursor: pointer;
		padding: 0;
		margin: 0 0 20rpx;
		color: #333;
		font-weight: 600;
		display: inline-block;
		font-size: 28rpx;
		border-color: #eee #ddd #ccc;
		line-height: 74rpx;
		border-radius: 20rpx;
		box-shadow: 0px 12rpx 18rpx -12rpx #999;
		background: linear-gradient(180deg, rgba(255,255,255,1) 0%, rgba(236,242,249,1) 50%);
		width: 100%;
		border-width: 2rpx;
		border-style: solid;
		text-align: center;
		height: 80rpx;
	}
	
	.category-three .tab {
		cursor: pointer;
		padding: 0;
		color: #fff;
		background: burlywood;
		display: inline-block;
		width: 100%;
		font-size: 28rpx;
		line-height: 80rpx;
		text-align: center;
	}
	
	.category-three .tab.active {
		cursor: pointer;
		padding: 0;
		color: #fff;
		background: #000;
		display: inline-block;
		width: 100%;
		font-size: 28rpx;
		line-height: 80rpx;
		text-align: center;
	}
</style>
